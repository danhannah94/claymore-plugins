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

## Phase 2: Red-team — fresh agents, grounded against reality

**The author is not the red-teamer.** A doc's author red-teaming their own work produces confidently-wrong findings — they share the blind spots that produced the gaps in the first place. (Proven live: an author's "nothing is deployed" HIGH finding was false; a fresh scout agent caught it by checking actual deployed state in one query.) So the strongest red-team is run by **fresh agents that verify the doc's claims against ground truth** — the real repo, live infra, actual deployed state — *not* against the doc's own assertions. Whenever the doc claims a fact ("X is deployed", "Y is missing", "Z commits behind"), a fresh agent checks it against reality before trusting it.

Two modes, matched to stakes (per the methodology's "match the process to the stakes"):

### (a) Fresh-agent fan-out — recommended for initiative / multi-epic docs (requires ultracode opt-in)

Run a `Workflow` (the fan-out-verify-synthesize primitive) that:

1. **Fans out N diverse lenses** as independent fresh agents — each attacks the doc through one angle and is blind to the others. A good default set:
   - **coverage** — what's missing for the real goal (rollback, comms, onboarding, abuse limits, "what does done mean")
   - **feasibility / execution** — can it actually be run? are the named dependencies real? verify deploy mechanics, infra state, tooling against the repo + live infra
   - **security** — isolation / data-loss / auth surface; is the threat-model coverage exhaustive against the *actual* code
   - **cost-reality** — do the cost / margin / performance claims hold against measured numbers
   - **verification-rigor** — does every acceptance criterion have a real, runnable check (see the standing lens below)
   - Each agent **grounds its findings in concrete evidence** (a `file:line`, a live query result) and marks anything speculative as such. It also flags the inverse: places the doc claims something ground truth *contradicts*.
2. **Adversarially verifies each finding** — an independent, skeptical agent re-checks it against ground truth, defaulting to **refuted** unless concrete evidence confirms it. This kills plausible-but-wrong findings before they reach the human (in practice this culls a meaningful fraction — e.g. a scary "cross-org data loss" finding that turned out to rest on planted test data).
3. **Synthesizes** the survivors into the triaged menu: deduped, grouped by severity, each with 2-3 resolution options (decision-per-minute style). Also lists the **refuted** findings briefly, so the human sees what was checked and discarded, not just what survived.

The synthesis IS the Phase-2 output. Carry it into Phase 3 triage.

### (b) Inline analysis — lightweight (small docs, or when ultracode isn't opted-in)

Do the same grounding yourself: read the repo / infra, **verify the doc's factual claims against reality**, then produce the severity-bucketed list. Same discipline, no fan-out — don't trust the doc's framing just because you're doing it solo.

### Severity buckets (both modes)

- **High** (architecturally significant / load-bearing): security issues, ambiguous component boundaries, unspecified contracts, decisions that compound across the codebase, gaps that block downstream epics, **a doc claim that ground truth contradicts**
- **Medium** (spec tightening): missing format specs, unspecified dependencies, vague test plans, unclear definitions-of-done, missing rationale for non-obvious choices
- **Low** (mention-and-move-on): minor naming, deferred items worth flagging, obvious-but-uncaptured assumptions
- **Cross-doc**: contradictions between sibling docs, broken handoffs between epics, scope drift, missing inter-epic dependencies
- **Verification-mechanism gaps** (standing lens): for every acceptance criterion / claim of "done", is there a **deterministic mechanism that checks it, and who owns it** — 🤖 AI-mechanized or 🧑 human-judgment? **A criterion with no checking mechanism is itself a finding.** If the mechanism doesn't exist, building it is in-scope work, not a nicety (see methodology: *build the feedback loop, not just the feature*).

Output the full list grouped by severity. **Don't ask permission to share — share it directly.** The user wants to see the gaps; they'll triage with you in Phase 3.

When presenting:
- Number each gap so they're addressable ("gap #3 above")
- Name the doc each gap is in
- Be specific about what's missing or wrong, not vague — cite the ground-truth evidence
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
