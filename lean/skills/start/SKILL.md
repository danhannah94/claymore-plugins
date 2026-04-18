---
name: start
description: "Bootstrap a new session. Reads next.md for context from the previous session, loads process.md for methodology, checks decisions.md for active decisions, and gives a brief orientation. Use at the beginning of every session. Accepts an optional project slug (e.g., /start quoteai) to scope context to a specific project directory."
argument-hint: "[project]"
---

# /start — Session Bootstrap

You are beginning a new session. Follow these steps in order.

## Step 0: Determine session scope

**If `$ARGUMENTS` is provided** (e.g., `/start quoteai`), treat it as a project slug. The project directory is `<workspace>/$ARGUMENTS/` where `<workspace>` is the current working directory at session start.

- **Project-scoped files live in the project directory:**
  - `<workspace>/$ARGUMENTS/next.md` — project-specific session handoff
  - `<workspace>/$ARGUMENTS/decisions.md` — project-specific active decisions
  - `<workspace>/$ARGUMENTS/CLAUDE.md` — project-specific AI instructions
- **Workspace-scoped files remain at the workspace root** and are read as fallback context for meta or cross-project work.

**If `$ARGUMENTS` is empty**, use the workspace root for all files (cross-project / meta sessions).

If the project directory doesn't exist yet:
- Check whether this is a fresh project (no directory + no Foundry design doc)
- If fresh: offer to bootstrap — create the project directory with `CLAUDE.md` (from `start/templates/claude-template.md`) and a starter `next.md` (from `start/templates/next-template.md`), then proceed
- If the directory is expected but missing, ask the human rather than silently failing

## Step 1: Load the methodology

Read `PROCESS.md` from the `process` skill directory (sibling plugin skill). Internalize the core principles — especially the three-file system and decisions lifecycle. This frames everything that follows.

## Step 2: Read `next.md`

Read `next.md` from the scope determined in Step 0:
- Project scope: `<workspace>/$ARGUMENTS/next.md`
- Workspace scope: `<workspace>/next.md`

This file was written by the AI at the end of the previous session. It contains:
- What was accomplished last session
- What's unfinished or next
- Context the previous session thought you'd need
- Specific instructions for picking up the work

**If `next.md` doesn't exist or is empty**, ask the human what they'd like to work on. This is a fresh start rather than a continuation.

## Step 3: Follow the instructions in `next.md`

It will typically tell you to:
- Read specific files or design docs for context
- Check the state of a branch or repo
- Verify that tools or services are available
- Start on specific work

## Step 4: Read `CLAUDE.md` if project-scoped

If running with a project slug, also read `<workspace>/$ARGUMENTS/CLAUDE.md` for project-specific instructions (conventions, commands, MCP setup, key pointers). If it doesn't exist, flag it in the orientation — we may want to create one.

## Step 5: Read `decisions.md`

Read `decisions.md` from the same scope as `next.md`:
- Project scope: `<workspace>/$ARGUMENTS/decisions.md`
- Workspace scope: `<workspace>/decisions.md`

Understand active decisions and what's been learned so far. If the file doesn't exist, that's fine — it's created as active decisions accumulate.

## Step 6: Check Foundry for context

If Foundry MCP tools (`mcp__foundry__*`) are available:
- Check for unresolved annotations on any design docs referenced by `next.md`
- Mention them in your orientation if found — they may be feedback waiting for `/reply`

## Step 7: Give a brief orientation

3-5 lines max:
- What you understand the current state to be (include project scope if applicable)
- What you plan to start on
- Any questions or gaps before you begin

Do not start work until the orientation is delivered and the user confirms.
