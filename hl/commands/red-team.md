---
description: Stress-test design docs by surfacing gaps grouped by severity, triaging them interactively, and applying locked decisions back to the docs. Accepts one or more Foundry doc paths.
argument-hint: "<doc-path>[,<doc-path>...]"
---

Red-team the following Foundry doc(s): **$ARGUMENTS**

Run the `red-team` skill. Five phases: Load (parallel reads of target docs +
unresolved annotations + related context) → Red-team (gaps grouped by severity,
grounded against the real repo/infra) → Triage (interactive AskUserQuestion rounds,
recommendation-first, batched 3-4 per round) → Apply (update_section calls
in one pass) → Recap (what landed, what's queued).

**The author is not the red-teamer.** Verify the doc's factual claims against
ground truth (repo, live infra, deployed state), not the doc's own assertions.
For initiative / multi-epic docs, prefer the **fresh-agent fan-out** mode — a
`Workflow` of diverse lenses that ground findings against reality, adversarially
verify each one, and synthesize the survivors (requires ultracode opt-in). For
small docs, do the same grounding inline. See the skill's Phase 2.

If no $ARGUMENTS provided, ask the user which doc(s) to red-team.

Don't ask permission to share the gap list — share it directly. Honest
tradeoff tables when the user pushes back on a recommendation.
