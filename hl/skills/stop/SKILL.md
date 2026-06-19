---
name: stop
description: "End a session cleanly under the hl methodology (v3). Triages decisions (active → decisions.md, settled → Foundry, demonstrated → prune), surfaces unresolved red-team gaps for the next session, then writes next.md for the next session to pick up cold. Use at the end of every session. Accepts an optional project slug (e.g., /hl:stop autri) to scope handoff to a specific project directory."
argument-hint: "[project]"
---

# /hl:stop — Session Handoff

The human is ending this session. Follow all five phases — don't skip the decisions triage or red-team gap surfacing. (Phase 4, Dev Memory, is autri-only and self-skips elsewhere.)

## Step 0: Determine session scope

**If `$ARGUMENTS` is provided** (e.g., `/hl:stop autri`), treat it as a project slug. All file writes are scoped to `<workspace>/$ARGUMENTS/`:
- `<workspace>/$ARGUMENTS/next.md` — project-specific session handoff
- `<workspace>/$ARGUMENTS/decisions.md` — project-specific active decisions

**If `$ARGUMENTS` is empty**, write to the workspace root (cross-project / meta sessions):
- `<workspace>/next.md`
- `<workspace>/decisions.md`

Use the same scope as the matching `/hl:start` that opened this session. If the project directory doesn't exist, create it before writing.

## Phase 1: Decisions triage

Review the session for decisions that were made. For each:

- **Active** (still being tested, might change) — add to `decisions.md` in the scoped location. Create the file if it doesn't exist.
- **Settled but non-obvious** (the reasoning matters long-term) — graduate to Foundry. Create an annotation on the relevant design doc section with the rationale using `mcp__foundry__create_annotation`. Then remove from `decisions.md` if it was there.
- **Demonstrated by the code** (the code and commit messages are sufficient) — prune from `decisions.md` if it was there. No further action needed.

If no decisions were made this session, skip this phase.

## Phase 2: Surface unresolved red-team gaps

If `/hl:red-team` (or any equivalent pressure-testing) ran this session and items were deferred, gather them up. Examples of deferral patterns:
- Spike-validation items in epic docs (e.g., "verify endpoint URL stability in Day 0 spike")
- Decisions explicitly postponed to the next session
- Open questions that surfaced but weren't triaged

These go into `next.md` under a dedicated "Deferred red-team gaps" section so `/hl:start` can surface them on the next bootstrap. Don't let them drop on the floor.

## Phase 3: Write `next.md`

Write `next.md` in the scoped location. The next session starts with zero memory of this conversation — `next.md` is the only bridge.

Structure it as direct instructions to the next AI session:

### Required sections:

**Context** — What to read first, in what order. Point at specific files, design docs, or Foundry pages. Don't summarize what's in them — just say what to read and why.

**Where we left off** — Concrete state: which repo, which branch, what's committed vs uncommitted, what's deployed vs local. Be specific — file paths, branch names, test counts.

**What was accomplished this session** — Brief list of what shipped or was built. Include key decisions made and why, especially non-obvious ones.

**Goal for next session** — What the human and AI agreed to do next, or what logically follows. Be specific about scope and boundaries.

**Deferred red-team gaps** — From Phase 2. Each gap should name what's open, why it was deferred, and what triggers picking it back up. Empty section is fine if no red-team ran or all gaps closed.

**Unresolved questions or risks** — Anything that came up but wasn't settled. Decisions that need human input. Things that might not work as expected.

### Rules:

- Write for an AI that has never seen this conversation — it can only read files, not remember chat
- Be specific: file paths, branch names, command examples, tool names
- Don't summarize file contents — point at the file and say "read this"
- Keep it under 80 lines for typical sessions; allow longer for substantial strategy/planning sessions
- If there are `decisions.md` updates needed, make them in Phase 1 before writing `next.md`
- If this is project-scoped, remind the next session to invoke `/hl:start $ARGUMENTS` (not bare `/hl:start`) to pick up project context

## Phase 4: Publish this session to Dev Memory (autri environments only)

The **write-side** of the dev-memory loop: this session becomes a retrievable episode so future sessions can recall its decisions (see the methodology principle *Consult Dev Memory* and the project's `autri-api` skill). It is an **autri-specific augmentation** — run it ONLY when the loop is configured, and skip silently otherwise (e.g. a work machine with no autri access; never let this block `/hl:stop`).

Do this AFTER `next.md` is written (Phase 3), so the episode captures the full session including the handoff. Guard on the dev-memory setup, then publish:

```bash
DEVMEM="${AUTRI_DEVMEMORY_DIR:-$HOME/Documents/Code/autri-platform/autri/dev-memory}"
if [ -f "$DEVMEM/.env" ]; then
  ( cd "$DEVMEM" && npx tsx publish-session.ts )   # add --transcript <this session's .jsonl> if you can resolve it
fi
```

`publish-session.ts` generates a hybrid/opus episode from the session transcript and **replace-uploads** it to the Dev Memory KB, deduped by session id — re-running `/hl:stop` (or a continued session) OVERWRITES the prior episode rather than duplicating. If `$DEVMEM/.env` is absent, SKIP this phase entirely; it is not part of the core methodology. Note the result (episode name + doc id, or "skipped — no Dev Memory") for Phase 5.

## Phase 5: Confirm

Tell the human:
- What you wrote in next.md (1-2 sentence summary) and where (project vs workspace scope)
- Any decisions.md or Foundry updates made in Phase 1
- Deferred red-team gaps that will surface next session (Phase 2)
- Whether the session was published to Dev Memory (Phase 4) — episode name + doc id, or that it was skipped
- Loose ends they should be aware of
- Remind them of the `/hl:start $ARGUMENTS` command if project-scoped
