---
name: red-team
description: "Stress-test design docs by surfacing gaps grouped by severity, triaging them interactively with the user, and applying locked decisions back to the docs. Use when reviewing epic plans, architecture docs, or any design that needs to be challenged before implementation. Faster than open-ended Foundry refinement — pace is 'decision-per-minute' not 'thread-per-session.' Targets Foundry docs by default."
argument-hint: "<doc-path>[,<doc-path>...]"
---

# /hl:red-team — Design Doc Stress Test

Pressure-test one or more design docs by surfacing gaps, triaging them interactively, and locking decisions back into the docs. Five phases.

## When to use

- Doc has been drafted but not yet implemented; you want to find gaps before you ship
- Decisions have been "soft-locked" (review threads agreeing) but haven't been tightened into the doc body
- About to start a multi-epic sprint and want to pressure-test the plan as a coherent whole
- Reviewing a sibling's draft before approval

## When NOT to use

- Doc is being created from scratch (use `/hl:draft` when it ships, or `/hl:start` workflow — red-team is for tightening, not authoring)
- You only have one specific concern (just ask directly)
- The doc is already battle-tested and only needs minor tweaks

## Inputs

`$ARGUMENTS` is a comma-separated list of Foundry doc paths, e.g.:

```
/hl:red-team projects/autri/epics/epic-1-agentcore-spike,projects/autri/epics/epic-2-library-connector
```

If `$ARGUMENTS` is empty, ask the user which doc(s) to red-team.

## Phase 1: Load (parallel reads)

In parallel:
- Load each target doc via `mcp__foundry__get_page`
- Load each target's unresolved annotations via `mcp__foundry__list_annotations` (status `submitted` and `replied`)
- Load any sibling docs that the targets reference (design.md, decisions.md, infra plans, etc.)
- Read project `CLAUDE.md` if present
- Read project `decisions.md` if present

Goal: have full context for the red-team pass in one round of tool calls.

## Phase 2: Red-team (silent analysis)

For each target doc, identify gaps in these severity buckets:

- **High** (architecturally significant / load-bearing): security issues, ambiguous component boundaries, unspecified contracts between modules, decisions that compound across the codebase, gaps that block downstream epics
- **Medium** (spec tightening): missing format specifications, unspecified dependencies, vague test plans, unclear definitions-of-done, missing rationale for non-obvious choices
- **Low** (mention-and-move-on): minor naming choices, deferred items worth flagging, obvious-but-uncaptured assumptions
- **Cross-doc**: contradictions between sibling docs, broken handoffs between epics, scope drift across the plan, missing dependencies between epics

Output the full list grouped by severity. **Don't ask permission to share — share it directly.** The user wants to see the gaps; they'll triage with you in Phase 3.

When presenting:
- Number each gap so they're addressable ("gap #3 above")
- Name the doc each gap is in
- Be specific about what's missing or wrong, not vague
- Lead with the most load-bearing items in each bucket

## Phase 3: Triage (interactive)

For each gap that needs a real decision, use `AskUserQuestion`:

- **Lead with your recommendation as option 1** (with `(Recommended)` suffix in the label)
- Provide 2-3 real alternatives with honest tradeoffs in the description
- **Batch 3-4 questions per `AskUserQuestion` call** — keep the cadence fast
- When user pushes back or asks "how big is this risk?", **give an honest tradeoff comparison with a table** if there are concrete numbers (cost, effort, risk severity) to compare
- Don't pad with obvious-fix items; if something is clearly correct once flagged, just apply it in Phase 4

Group questions by domain when batching (e.g., "Round 1: EPIC-2 architectural decisions" → "Round 2: spec tightening" → "Round 3: minor cleanups"). Each `AskUserQuestion` call should feel like a coherent topic.

If a user answer reveals a follow-up question or factual gap (e.g., "wait, does X exist in our codebase?"), pause and check before continuing the triage round.

## Phase 4: Apply (single pass)

Once all decisions are locked, update doc sections via `mcp__foundry__update_section`. For each updated section:

- Reflect locked decisions in the body (not in comments, not as deferrals — actually rewrite the relevant prose)
- For each touched doc, add or update a **"Locked this triage pass (YYYY-MM-DD)"** sub-section under "Notes / open questions"
- Keep "Still open" items explicitly distinct from locked ones
- Update any related docs (e.g., if EPIC-2 schema decision affects EPIC-3, update both)

Make the updates in parallel where possible. Don't update the doc after every decision — batch at the end so the conversation stays in the architectural lane.

## Phase 5: Recap

Brief summary (under 200 words):
- **What landed** — which docs got updated, which decisions locked
- **Open follow-ups for the human** — action items they need to handle in the real world (e.g., "open AWS account", "register Google OAuth app")
- **Deferred red-team gaps** — items consciously punted, with the trigger for picking them back up
- **Queued for next session** — one sentence

Use markdown headings for each section. Keep it scannable.

## Style notes

- **Honest tradeoff comparisons** when the user pushes back. Don't just defend your recommendation; lay out the actual tradeoff. Tables help when there are numbers.
- **Don't ask "should I share the gap list?"** Share it.
- **Recommendation-first** in every question — it's faster for the user to ratify or push back than to evaluate equal-weight options.
- **Track progress with TodoWrite** for the phases — this is a multi-step skill and the todo list keeps the user informed of where you are.
- **Apply doc updates at the end, not after each decision** — keeps the conversation flowing.
