---
description: Ship a batch of stories through the agentic pipeline — dependency waves, parallel feature agents in worktrees, conditional QA, optional deploy. The methodology's Ship stage.
argument-hint: "[design doc with Story Summary, or inline batch spec]"
---

Ship this batch: **$ARGUMENTS**

Run the `ship` skill. Before Phase 0, confirm the three kickoff knobs with the
user (QA backend: test-env+Crucible vs headless-preview+dev-auth; merge
authority: human-merge vs delegated; agent model: inherit/Sonnet vs Opus),
detecting sensible defaults from the project.

Then: parse the batch → compute dependency waves (topo sort) → present the wave
plan and WAIT for approval → execute each wave (parallel feature agents in git
worktrees → orchestrator code review → conditional QA → wave review/merge →
rebase remaining → wave gate) → final report. Only run the optional deploy +
migrate phase (2.5) if the user asked to ship to prod this run — and never
improvise a prod migration; discover the real path first.

Assumes stories are already refined with explicit acceptance criteria (this is
downstream of /hl:red-team and /hl:blue-team). If a story is ambiguous mid-run,
stop and refine — do not have agents guess.
