---
name: blue-team
description: "Make scope cuts from a red-team menu and produce the requirements doc. Where red-team attacks the problem and pokes holes, blue-team defends time and resources by deciding what's actually in scope. Use after /hl:red-team has surfaced the full landscape. The output is the requirements contract that grounds the rest of the initiative."
argument-hint: "<initiative-name>"
---

# /hl:blue-team — Scope Cuts → Requirements Doc

Red-team builds the menu. Blue-team cuts from it. The output is the requirements contract that everything downstream references.

## When to use

- After `/hl:red-team` has surfaced a full landscape of gaps + scope items + alternatives across drafted epics
- When starting a multi-epic initiative and you need an explicit scope contract before implementation
- When scope creep is suspected — re-run blue-team to formally amend the requirements

## When NOT to use

- For single-epic or single-task work (overkill — the epic's Scope section is enough)
- Before red-team has run (you need the menu before you can cut from it)
- For exploratory spikes where requirements are intentionally fluid

## Inputs

`$ARGUMENTS` is the initiative name, e.g.:

```
/hl:blue-team autri-beta
```

Used for the requirements doc filename and the L0 goal statement.

## Phase 1: Aggregate the menu

Pull the full landscape from existing sources, in this priority order:

1. **Prior red-team output** (from this session or `next.md` deferred gaps) — if available, this is the primary source
2. **Epic doc Scope sections** across all epics in the initiative — scope items become L2/L3 requirement candidates
3. **Epic doc Risks + Open Questions** — risks become potential requirements (mitigations); open questions become scope decisions
4. **Cross-cutting items** that surfaced in red-team but didn't fit cleanly in one epic (email infra, auth, monitoring, feedback)
5. **Architecture decisions** (`decisions.md`) — locked decisions are constraints, not requirements, but they shape what's possible
6. **Out-of-scope items already documented** in the epic docs — these come into the requirements doc as explicit `out-of-scope` entries

Goal: full landscape in one place before any cutting begins. Skip cuts at this phase — even items you think are obviously out get listed.

## Phase 2: Initial sort

AI proposes a must/should/nice/out-of-scope status for each item. Use these criteria:

- **must-have** — the initiative goal (L0) literally cannot be delivered without this
- **should-have** — strongly desired; affects user value or operational integrity, but the initiative could ship with it cut
- **nice-to-have** — polish or convenience; cleanly defer to a follow-up
- **out-of-scope** — explicitly NOT in this initiative (this is the scope-discipline anchor)

Group by L1 (high-level requirement / epic-shape). Within each L1, list L2 requirements with proposed status + which epic implements it. Present the grouped landscape with stats: `N must / N should / N nice / N out-of-scope`.

**Don't ask permission to share — share it directly.** The user wants to see the menu so they can adjust it.

## Phase 3: Triage (interactive)

For each item where the priority is genuinely uncertain (skip obvious must-haves and obvious out-of-scopes), use `AskUserQuestion`:

- **Lead with your recommendation as option 1** (with `(Recommended)` suffix in the label)
- Provide 2-3 real alternatives — usually `keep at <current status>`, `cut to <lower>`, `promote to must-have`
- **Batch 3-4 questions per `AskUserQuestion` call** — keep cadence fast
- When user pushes back or proposes a meaningful change (e.g., "actually cut all of X"), pause and address the implication honestly — with a tradeoff table if there are real numbers to compare
- Be ready for mid-triage scope pivots; they're feature, not bug

Don't waste questions on items that are obviously in or out. Save them for the genuine should-vs-must and should-vs-cut judgment calls.

## Phase 4: Write the requirements doc

Create the doc at `projects/<project>/requirements/<initiative>.md` via `mcp__foundry__create_doc`. Structure:

- **L0 — Initiative goal** (one paragraph): what we're shipping, who for, by when, what "done" means
- **L1.N sections** (one per high-level requirement, epic-shape): each contains a table of L2 requirements with `ID | Requirement | Status | Solution` columns
- **Beta user / persona summary** (if applicable)
- **DDoS / security defense** (if applicable; cross-cutting)
- **Explicitly OUT OF SCOPE** section — the load-bearing artifact. Group by category for readability. This is what prevents future scope creep.
- **Amendments** table (empty at drafting; future scope changes get logged here with date + reason)
- **Cross-references** to architecture decisions, design docs, epic implementations
- **Definition of "<initiative> is done"** — explicit success criteria

Use `R<L1>.<L2>` IDs (e.g., `R1.3.4`) so requirements are addressable in future discussion and red-team can reference them.

## Phase 5: Recap + reconciliation

Brief summary (under 200 words):
- **What landed** — link to the requirements doc, headline stats (N must / N should / N out-of-scope)
- **Notable scope cuts** — anything substantial that got cut during triage (the "we decided not to" items worth flagging)
- **Notable scope pivots** — if the triage surfaced an unexpected reframe (e.g., "cut all email, replace with in-app notifications")
- **Epic doc reconciliation needed** — if the requirements contract diverged from the existing epic drafts, list what needs updating. Offer to apply now or defer.
- **Queued for next session** — one sentence

If the contract diverged from epic drafts, **always reconcile** — either by updating epics now or by explicitly punting with a tracked todo. Don't leave the contract and the implementation drifting silently.

## Style notes

- **Recommendation-first** in every question
- **Honest tradeoff tables** when scope-cutting is non-obvious (cost / effort / risk comparisons)
- **Don't ask "should I share the menu?"** Share it
- **Track progress with TodoWrite** for the phases — multi-step skill, todos keep the user oriented
- **Mid-triage scope pivots are normal** — accommodate them gracefully, don't force the user back into the original frame
- **The out-of-scope section is the load-bearing artifact** — write it carefully. Group by category. Each entry should be specific enough that re-introduction is a real decision, not a vague "maybe later"
