# Foundry MCP Patterns & Recipes

Concrete, copy-pasteable recipes for the workflows that come up repeatedly. If you're doing one of these for the first time, read the matching recipe end-to-end before calling a tool.

## Recipe 1: Safe NEXT.md update (session handoff)

**When:** wrapping a CSDLC session. See `csdlc-session-end` for the content structure; this recipe is the mechanical safe-write procedure.

**Why special:** NEXT.md is a running journal — prior `✅ SHIPPED` sections must be preserved verbatim. Getting it wrong silently drops history.

```
1. current = get_page('status/next.md')
2. Extract the H1 body from current (everything after the '# NEXT.md ...' heading).
3. Compose new_body:
   - New 🎯 Priority Stack (refreshed)
   - New 🔴 NEXT section
   - New ✅ SHIPPED — <today> block, freshly written
   - ALL prior ✅ SHIPPED blocks from current, byte-for-byte unchanged
   - 🟡 ON DECK (refreshed)
   - 📋 Known Tech Debt (with checkboxes updated)
4. delete_doc('status/next.md')
5. create_doc('status/next.md', template='blank')
6. update_section(
     path='status/next.md',
     heading_path='# NEXT.md — What We\'re Working On',
     content=new_body   # H2+ only — Foundry auto-parses children
   )
7. verify = get_page('status/next.md')
8. Eyeball: prior SHIPPED sections still present? Priority Stack correct? No [2] suffixes?
```

**Failure modes:**
- If step 6 returns 404 with `available_headings`, you probably typed the H1 wrong. Check step 5 landed — `get_page` again and confirm the H1 exists.
- If step 8 shows `[2]` duplicates, a retry on step 6 happened after a partial success. Delete and redo from step 4.
- Never retry step 6 surgically — always restart from step 4.

## Recipe 2: Surgical leaf edit that preserves annotations

**When:** a page has live annotations you need to preserve, and you only need to change one H2 or H3 section.

**Key rule:** update the **deepest** heading you can. Updating a parent re-parses its children and may drop their annotations. Leaf updates only affect the leaf.

```
1. current = get_page(path)
2. Find the exact heading_path to the leaf: '# H1 > ## H2 > ### H3 (target)'
3. list_annotations(path) — note which sections have annotations.
4. If the target leaf has annotations you care about, STOP — any update will drop them.
   Options:
     a) Reply to the annotation first (create_annotation with parent_id), then resolve it.
     b) Archive/export the annotation content, then proceed.
5. update_section(path, heading_path=<leaf>, content=<new leaf body>)
6. list_annotations(path) — confirm unrelated annotations survived.
```

**Gotcha:** `update_section` on an H1 with a full-body string behaves like a rewrite and will drop all child annotations. Treat H1 updates as rewrites, not surgical edits.

## Recipe 3: Recovering from corrupted state

**When:** a page has `[2]` or `[3]` suffixes on sections, duplicate H2s, orphaned content at the wrong level — usually from a pre-fix `update_section` that silently appended, or from a retry after partial failure.

```
1. current = get_page(path)
2. Read the raw markdown. Identify which sections are canonical and which are duplicates.
3. Compose clean_body locally (markdown string).
4. If the page has no annotations worth keeping:
     a) delete_doc(path)
     b) create_doc(path, template='blank')
     c) update_section(path, heading_path='# <H1>', content=clean_body_without_h1)
   Else:
     a) For each corrupted leaf, update_section on that specific leaf.
     b) delete_section on the duplicate leaves (use the [2]/[3] heading_path).
     c) Verify with get_page.
5. get_page(path) again and diff against clean_body by eye.
```

**Prevent next time:** never retry a failed `update_section` without first re-reading the page. A silent append is worse than a loud failure.

## Recipe 4: Dispatching a sub-agent via the ticket doc

**When:** the AI Lead has a refined ticket and is handing it to a Claude Code sub-agent through the foundry-cc-bridge. See `csdlc-craft-agent-prompt` for the prompt content; this recipe is the Foundry-side mechanics.

```
1. Refine the ticket in its Foundry doc (projects/<project>/epics/<epic>.md#<story>).
2. Compose the agent prompt (follow csdlc-craft-agent-prompt).
3. The bridge spawns the sub-agent with the prompt.
4. Sub-agent read path: get_page on the ticket doc, search_docs for related context,
   list_annotations on the ticket doc to see refinement history.
5. Sub-agent completion path: create_annotation on the ticket doc with a status report.
   (Sub-agents have no write access to docs, only annotations.)
6. AI Lead polls annotations on the ticket doc for the completion report.
7. On acceptance: resolve_annotation. On rework: create_annotation (reply) with
   parent_id pointing at the sub-agent's report.
```

**Rules:**
- Sub-agents can read private docs via the MCP. Don't assume doc visibility is a security boundary for them.
- Sub-agents cannot create or edit docs, only annotations. If a ticket requires doc authorship, the AI Lead handles it post-acceptance.
- The ticket doc is the canonical source of truth. Anything important must live in the doc or an annotation on it, not in the prompt alone.

## Recipe 5: Reviewing a draft doc with annotations

**When:** a design doc or draft needs inline comments from the AI Lead before being promoted.

```
1. get_page(draft_path)
2. For each comment:
     create_annotation(
       path=draft_path,
       heading_path='# H1 > ## <section>',
       body='<comment>',
       state='submitted'
     )
3. Optional: submit_review to batch them into a thread.
4. User addresses comments, replies with create_annotation(parent_id=<original_id>).
5. resolve_annotation per comment as it's addressed.
```

**Never:** delete_doc or H1-update_section on a doc mid-review. Annotations are lost.

## Recipe 6: Reading a doc tree for orientation

**When:** catching up on a project area without knowing exact paths.

```
1. list_pages(path_prefix='projects/<project>/') — get the tree shape.
2. For each interesting page: get_page to read.
3. search_docs for keywords if the tree is too big to read linearly.
4. For any page that looks actively discussed: list_annotations to see current threads.
```

**Efficiency:** `list_pages` is cheap, `get_page` is cheap, but reading 20 full pages isn't. Use `search_docs` to narrow before pulling full content.

## Anti-patterns (do not do these)

- **Retrying `update_section` on a heading_path that 404'd without re-reading the page.** If it 404'd pre-fix, it may have silently appended. Always `get_page` first.
- **Calling `delete_doc` on a page with live annotations to "reset" it.** You lose the annotations, and Foundry doesn't tell you whose they were until they're gone.
- **Updating the H1 when you meant to update an H2.** H1 updates are page-wide rewrites. Always target the deepest heading you can.
- **Passing the H1 heading inside the `content` body of `update_section`.** Foundry will nest it as an H2 of itself, creating a broken tree. The `content` should start at H2 when updating the H1.
- **Using `search_docs` for sensitive private content without confirming the leak bug is fixed.** Check first, or use `list_pages` + `get_page` on known paths instead.
- **Treating annotations as durable.** They're cascade-deleted on rewrite. Copy the content out if you need it long-term.
