---
name: foundry-mcp-reference
description: Reference for the Foundry MCP toolset — the mental model, nested-heading path format, known pitfalls, and safe editing patterns. Use whenever you're about to call a Foundry MCP tool and aren't sure of the approach, whenever you hit a weird Foundry error (schema mismatch, heading-not-found), or whenever the user asks how to edit/write to Foundry. Tool parameter shapes are self-described by the MCP server — this skill covers the things the tool docstrings don't.
---

# Foundry MCP Reference

Foundry is Dan's self-hosted doc system (deployed at `https://foundry-claymore.fly.dev`, MCP endpoint at `/mcp/sse`). It exposes tools for reading, writing, annotating, and reviewing docs. Tool parameter shapes and return types are self-described by the MCP server itself — this skill covers what the tool docstrings can't: the mental model, multi-tool workflows, and the pitfalls that only show up in practice.

## Mental model

Pages are identified by path (e.g., `status/next.md`). Each page has a section tree: H1 at root, H2s as children, H3s below. Tools operate on whole pages (`get_page`, `create_doc`, `delete_doc`) or individual sections (`get_section`, `update_section`, `insert_section`, `delete_section`). Annotations are per-section comments with a state machine (submitted → resolved); reviews batch annotations into review threads. There's also `search_docs`, `sync_to_github`, and admin tools you'll rarely touch.

## Three things that bite everyone

1. **Nested heading paths use `>` with `#` on every level.** Format: `# Page H1 > ## H2 > ### H3`. Miss a `#` and you'll get a 404 (with `available_headings` in the response to help you correct). Param is `heading_path`; insert uses `after_heading_path`.

2. **`update_section` throws strict 404s with `available_headings` on a heading miss.** Don't panic if the first call misses — read the `available_headings` field in the error response and retry with the correct path. (Historically this silently appended duplicate sections, creating `[2]` suffixes. If you encounter pre-fix corruption, clean up with the rewrite pattern.)

3. **`search_docs` has leaked private docs in the past.** Verify the bug status before querying anything touching private content. `auth_token` on `search_docs` unlocks private results *on top of* an authenticated connection — it does not bootstrap the connection itself.

## The safe-edit workflow

1. `get_page(path)` — capture current state.
2. Decide: surgical (one leaf section) or rewrite (multiple sections).
3. **Surgical on a leaf (H2/H3):** `update_section` with the full `heading_path`. Fast and cheap when you're touching one section. **Preserves annotations on sibling sections.**
4. **Rewrite:** `delete_doc` + `create_doc(template='blank')` + one `update_section` on the H1 with the full body. Foundry parses embedded H2/H3 headings in the content into child sections on save, so you get the full structure in one transaction. **Warning: this cascades and deletes every annotation on the page.** Only rewrite when there are no annotations you care about.
5. For sensitive edits (canonical `next.md`, active design drafts), `get_page` again after and diff the old/new by eye.

Pays off fast: a corrupted page takes much longer to recover from than the verification step takes.

## When to rewrite vs. surgical

- **Rewrite** when: changing 3+ sections, reordering sections, writing a new session handoff in `next.md`, recovering from corrupted state — AND you don't care about existing annotations on the page.
- **Surgical on a leaf** when: patching a typo, updating one "Open Questions" section, appending to a changelog, anything where the rest of the doc is stable, OR when the page has live annotations you need to preserve.
- **Never surgical on NEXT.md** — too many sections change per update, atomic rewrite is cleaner (and NEXT.md doesn't accumulate annotations).

## NEXT.md is special: journal, not bootloader

`status/next.md` is a running journal where `✅ SHIPPED` sections accumulate across sessions. **Never delete prior SHIPPED blocks** when updating it. The rewrite pattern above still applies — the key is that the `content` string you pass to `update_section` must include every prior SHIPPED section unchanged. Always `get_page` first to capture the existing content, then splice in the new session's work. See the `csdlc-session-end` skill for the full structure.

## Annotations and reviews

Annotations are comments attached to a specific section (via `heading_path`) with a state machine: `draft` → `submitted` → `resolved`. Sub-agents dispatched via the foundry-cc-bridge use annotations as their primary write channel — the AI Lead polls annotations on the ticket doc to get completion reports. Reviews batch annotations into threads that can be submitted together.

Key operations: `create_annotation`, `list_annotations`, `resolve_annotation`, `reopen_annotation`, `submit_review`. To reply to an annotation, create a new annotation with `parent_id` set to the original's ID.

**Annotation loss is easier than you'd think.**
- `delete_doc` cascades and drops every annotation on the page.
- `update_section` on the **H1** with a full body also appears to drop annotations on child sections — Foundry re-parses the embedded H2/H3 tree on save, and the children are effectively replaced.
- `update_section` on a **leaf (H2/H3)** preserves annotations on *sibling* sections but will drop annotations on the specific section being replaced.

If a page has live annotations you need to preserve, don't rewrite it. Use the narrowest surgical update that avoids the annotated sections, or resolve/archive the annotations first.

## Auth

Two layers: **connection auth** (MCP handshake at connector setup) and **per-request auth** (`auth_token` param on some tools). Per-request tokens cannot bootstrap a broken connection — if the connector is unauthenticated, every call fails regardless of what you pass. "Requires authentication" errors mean reconnect the MCP in Cowork connectors, not pass a token.

Hosted connectors (Claude.ai, Cowork) require OAuth 2.0 + dynamic client registration per the MCP Authorization spec. Static bearer tokens only work for clients you control (stdio bridge, fetch-based transports). Full OAuth support is tracked under the E12 epic.

## See also

- `references/patterns.md` — concrete recipes for common workflows (safe NEXT.md update, annotation-preserving edits, recovering from corrupted state, dispatching a sub-agent via annotations).
