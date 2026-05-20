---
description: Stress-test design docs by surfacing gaps grouped by severity, triaging them interactively, and applying locked decisions back to the docs. Accepts one or more Foundry doc paths.
argument-hint: "<doc-path>[,<doc-path>...]"
---

Red-team the following Foundry doc(s): **$ARGUMENTS**

Run the `red-team` skill. Five phases: Load (parallel reads of target docs +
unresolved annotations + related context) → Red-team (silent analysis,
gaps grouped by severity) → Triage (interactive AskUserQuestion rounds,
recommendation-first, batched 3-4 per round) → Apply (update_section calls
in one pass) → Recap (what landed, what's queued).

If no $ARGUMENTS provided, ask the user which doc(s) to red-team.

Don't ask permission to share the gap list — share it directly. Honest
tradeoff tables when the user pushes back on a recommendation.
