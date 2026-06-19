---
name: start
description: "Bootstrap a new session under the hl methodology (v3). Reads next.md for context from the previous session, loads methodology.md for principles, checks decisions.md for active decisions, surfaces unresolved red-team gaps from the prior session, and gives a brief orientation. Use at the beginning of every session. Accepts an optional project slug (e.g., /hl:start autri) to scope context to a specific project directory."
argument-hint: "[project]"
---

# /hl:start — Session Bootstrap

You are beginning a new session under the Hannah Labs v3 methodology. Follow these steps in order.

## Step 0: Determine session scope

**If `$ARGUMENTS` is provided** (e.g., `/hl:start autri`), treat it as a project slug. The project directory is `<workspace>/$ARGUMENTS/` where `<workspace>` is the current working directory at session start.

- **Project-scoped files live in the project directory:**
  - `<workspace>/$ARGUMENTS/next.md` — project-specific session handoff
  - `<workspace>/$ARGUMENTS/decisions.md` — project-specific active decisions
  - `<workspace>/$ARGUMENTS/CLAUDE.md` — project-specific AI instructions
- **Workspace-scoped files remain at the workspace root** and are read as fallback context for meta or cross-project work.

**If `$ARGUMENTS` is empty**, use the workspace root for all files (cross-project / meta sessions).

If the project directory doesn't exist yet:
- Check whether this is a fresh project (no directory + no Foundry design doc)
- If fresh: offer to bootstrap — create the project directory with a starter `CLAUDE.md` and `next.md`, then proceed
- If the directory is expected but missing, ask the human rather than silently failing

## Step 1: Load the methodology

Read `METHODOLOGY.md` from the `methodology` skill directory (sibling plugin skill). Internalize the core principles — especially the pipeline (`requirements → draft → red-team → reply → update → ship`), the three-file system, and the decisions lifecycle. This frames everything that follows.

## Step 2: Read `next.md`

Read `next.md` from the scope determined in Step 0:
- Project scope: `<workspace>/$ARGUMENTS/next.md`
- Workspace scope: `<workspace>/next.md`

This file was written by the AI at the end of the previous session. It contains:
- What was accomplished last session
- What's unfinished or next
- Context the previous session thought you'd need
- **Unresolved red-team gaps** (if `/hl:red-team` ran last session and items were deferred)
- Specific instructions for picking up the work

**If `next.md` doesn't exist or is empty**, ask the human what they'd like to work on. This is a fresh start rather than a continuation.

## Step 3: Read `CLAUDE.md` if project-scoped

If running with a project slug, also read `<workspace>/$ARGUMENTS/CLAUDE.md` for project-specific instructions (conventions, commands, MCP setup, key pointers). If it doesn't exist, flag it in the orientation — we may want to create one.

## Step 4: Read `decisions.md`

Read `decisions.md` from the same scope as `next.md`:
- Project scope: `<workspace>/$ARGUMENTS/decisions.md`
- Workspace scope: `<workspace>/decisions.md`

Understand active decisions and what's been learned so far. If the file doesn't exist, that's fine — it's created as active decisions accumulate.

## Step 5: Check Foundry for context

If Foundry MCP tools (`mcp__foundry__*`) are available:
- Check for unresolved annotations on any design docs referenced by `next.md` (status `submitted` or `replied`)
- Mention them in your orientation if found — they may be feedback waiting for `/hl:reply` (or `/lean:reply` until hl:reply ships)

## Step 6: Surface deferred red-team gaps

If `next.md` includes a "Deferred red-team gaps" section (written by a prior `/hl:stop`), summarize the most load-bearing ones in your orientation. These are items that came up in a previous red-team but weren't resolved — they're often the right thing to start with.

## Step 7: Recall from Dev Memory (autri environments only)

The **read-side** of the dev-memory loop — pull prior rationale BEFORE orienting so a cold session starts with its own history, not a blank slate (see the methodology principle *Consult Dev Memory* and the project's `autri-api` skill). Autri-specific; skip silently where it isn't configured. Guard on the setup:

```bash
DEVMEM="${AUTRI_DEVMEMORY_DIR:-$HOME/Documents/Code/autri-platform/autri/dev-memory}"
[ -f "$DEVMEM/.env" ] && echo "dev-memory available"
```

If available, read `$DEVMEM/.env` for AUTRI_API_URL / AUTRI_API_KEY / AUTRI_DEVMEMORY_KB, then recall two ways (run `autri` from the autri repo root with those env vars set):
- **Recent** — `pnpm autri filter <AUTRI_DEVMEMORY_KB> --rank recency --recency-field date --k 8`: the last sessions' decisions ("where were we").
- **On-topic** — if `next.md` names a clear focus, `pnpm autri search <AUTRI_DEVMEMORY_KB> "<that focus>"`: prior rationale on it.

Fold the 2-3 most relevant prior decisions into the orientation (Step 8). If `$DEVMEM/.env` is absent, SKIP — no recall, no error.

## Step 8: Give a brief orientation

3-5 lines max:
- What you understand the current state to be (include project scope if applicable)
- What you plan to start on (or propose starting on)
- Any deferred red-team gaps worth surfacing first
- The most relevant prior decisions recalled from Dev Memory (Step 7), if any
- Any questions or gaps before you begin

Do not start work until the orientation is delivered and the user confirms.
