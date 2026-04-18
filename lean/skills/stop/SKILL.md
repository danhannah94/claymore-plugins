---
name: stop
description: "End a session cleanly. Triages decisions (active → decisions.md, settled → Foundry, demonstrated → prune), then writes next.md for the next session to pick up cold. Use at the end of every session. Accepts an optional project slug (e.g., /stop quoteai) to scope handoff to a specific project directory."
argument-hint: "[project]"
---

# /stop — Session Handoff

The human is ending this session. Follow all three phases — don't skip the decisions triage.

## Step 0: Determine session scope

**If `$ARGUMENTS` is provided** (e.g., `/stop quoteai`), treat it as a project slug. All file writes are scoped to `<workspace>/$ARGUMENTS/`:
- `<workspace>/$ARGUMENTS/next.md` — project-specific session handoff
- `<workspace>/$ARGUMENTS/decisions.md` — project-specific active decisions

**If `$ARGUMENTS` is empty**, write to the workspace root (cross-project / meta sessions):
- `<workspace>/next.md`
- `<workspace>/decisions.md`

Use the same scope as the matching `/start` that opened this session. If the project directory doesn't exist, create it before writing.

## Phase 1: Decisions triage

Review the session for decisions that were made. For each:

- **Active** (still being tested, might change) — add to `decisions.md` in the scoped location. Create the file if it doesn't exist.
- **Settled but non-obvious** (the reasoning matters long-term) — graduate to Foundry. Create an annotation on the relevant design doc section with the rationale using `mcp__foundry__create_annotation`. Then remove from `decisions.md` if it was there.
- **Demonstrated by the code** (the code and commit messages are sufficient) — prune from `decisions.md` if it was there. No further action needed.

If no decisions were made this session, skip this phase.

## Phase 2: Write `next.md`

Write `next.md` in the scoped location. The next session starts with zero memory of this conversation — `next.md` is the only bridge.

If a template exists at `start/templates/next-template.md` and the target `next.md` is brand new, use the template as a skeleton.

Structure it as direct instructions to the next AI session:

### Required sections:

**Context** — What to read first, in what order. Point at specific files, design docs, or Foundry pages. Don't summarize what's in them — just say what to read and why.

**Where we left off** — Concrete state: which repo, which branch, what's committed vs uncommitted, what's deployed vs local. Be specific — file paths, branch names, test counts.

**What was accomplished this session** — Brief list of what shipped or was built. Include key decisions made and why, especially non-obvious ones.

**Goal for next session** — What the human and AI agreed to do next, or what logically follows. Be specific about scope and boundaries.

**Unresolved questions or risks** — Anything that came up but wasn't settled. Decisions that need human input. Things that might not work as expected.

### Rules:

- Write for an AI that has never seen this conversation — it can only read files, not remember chat
- Be specific: file paths, branch names, command examples, tool names
- Don't summarize file contents — point at the file and say "read this"
- Keep it under 80 lines. Dense and actionable, not narrative
- If there are `decisions.md` updates needed, make them in Phase 1 before writing `next.md`
- If this is project-scoped, remind the next session to invoke `/start $ARGUMENTS` (not bare `/start`) to pick up project context

## Phase 3: Confirm

Tell the human:
- What you wrote in next.md (1-2 sentence summary) and where (project vs workspace scope)
- Any decisions.md or Foundry updates made in Phase 1
- Loose ends they should be aware of
- Remind them of the `/start $ARGUMENTS` command if project-scoped
