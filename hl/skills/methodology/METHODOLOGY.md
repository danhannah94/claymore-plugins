# Hannah Labs Methodology (v3)

Third iteration of a collaborative-development methodology. Previously OpenClaw, then lean. Same goal: **low ceremony, high signal.** Only load-bearing principles — no rituals.

## The pipeline

```
draft → red-team → blue-team → (epic refinement) → reply → update → ship
```

Each stage has a defined output and a clear hand-off to the next. Stages are skippable when stakes are low.

| Stage | Skill | Output | Skip when |
|---|---|---|---|
| Draft | manual (no skill yet) | Rough doc(s) in Foundry — epic drafts, plans | You're editing an existing doc, not creating one |
| Red-team | `/hl:red-team` | Gaps surfaced + decisions locked into doc bodies | Doc is already battle-tested; only minor tweaks needed |
| Blue-team | `/hl:blue-team` | Requirements doc (the scope contract) | Single-epic or single-task work |
| Epic refinement | manual | Epic docs reconciled with requirements contract | No divergence between epics and requirements |
| Reply | `/hl:reply` (planned; currently `/lean:reply`) | AI replies to human feedback in Foundry annotations | No outstanding feedback |
| Update | `/hl:update` (planned; currently `/lean:update`) | Agreed decisions locked into doc body; threads resolved | No agreed-upon decisions pending |
| Ship | `/hl:ship` | Code that implements the design — batch/wave pipeline (parallel feature agents in worktrees, conditional QA, optional deploy) | Single trivial change (implement it directly) |

**Use the full pipeline for multi-epic initiatives. Skip stages aggressively for one-off work.** A bug fix doesn't need blue-team. A button addition doesn't need red-team. Match the process to the stakes.

## Why red-team comes before blue-team

**Subtraction is easier than addition.** Red-team surfaces a comprehensive landscape of what could be built (the menu). Blue-team then makes scope cuts from a position of total visibility. You can't write good requirements for something you've never imagined — but you CAN imagine the landscape adversarially, then prune.

This is the divergent-then-convergent pattern. Red-team is divergent (yes-and). Blue-team is convergent (no-because).

## The two pillars

- **Red team attacks** — adversarial review of drafted docs. Pokes holes, surfaces gaps, builds the menu.
- **Blue team defends** — defends time and resources by cutting scope. Produces the requirements contract.

The pair maps loosely to security red/blue teams but works equally as: red surfaces everything possible; blue locks in what's actually worth building.

## The three-file system

| File | Scope | Who writes | Purpose |
|---|---|---|---|
| `methodology.md` (this doc) | Plugin | Both (iterate together) | How we work |
| `next.md` | Workspace or project | AI (via `/hl:stop`) | Session handoff |
| `decisions.md` | Per-project | AI (during work) | Active decisions not obvious from the code |

Plus, per-initiative:

| File | Scope | Who writes | Purpose |
|---|---|---|---|
| `projects/<project>/requirements/<initiative>.md` | Per-initiative | AI (via `/hl:blue-team`) | The scope contract for an initiative |

## Core principles

**The code is the primary artifact.** If architecture isn't legible from the code, fix the code, not the docs.

**Foundry captures WHY. Code is WHAT.** Design docs and annotations record rationale, trade-offs, rejected alternatives. Don't duplicate code in prose.

**Scope before building, design while building.** Agree on the scope contract (requirements doc) before starting. Make design decisions during implementation as real constraints surface.

**Refine when uncertain.** Surface low-confidence or high-stakes decisions to the human. Don't guess. This valuable friction prevents confidently building the wrong thing.

**Red-team before implementing. Blue-team before committing to scope.** Pressure-test the plan against gaps. Then explicitly cut scope before engineering time is committed. Cheaper to debate the contract than rewrite code.

**Amendments are visible.** Requirements doc has an `Amendments` table. When scope changes mid-sprint (and it will), log it there — not silently. Scope creep that's documented is feature; scope creep that's silent is bug.

**Commit messages capture WHY.** A commit message that says what changed is useless — the diff shows that. Write why, what was considered and rejected, and any non-obvious reasoning.

## Decisions lifecycle

| State | Location | Trigger |
|---|---|---|
| Active | `decisions.md` | Being tested, might change |
| Settled but non-obvious | Foundry annotation on the relevant design doc | Reasoning matters long-term |
| Demonstrated by the code | Pruned from `decisions.md` | Code + commits are sufficient |

Keep `decisions.md` short — ideally 10-20 active entries. If it's growing unbounded, entries aren't graduating or being pruned.

## Requirements lifecycle

| State | Trigger |
|---|---|
| Drafted | `/hl:blue-team` creates the doc |
| Amended | Scope change mid-sprint — logged in the `Amendments` table |
| Satisfied | All `must` requirements met; initiative is "done" per the contract |
| Archived | Initiative shipped; doc kept as historical reference |

The requirements doc is the durable artifact. Epic docs evolve with implementation; requirements stay stable (with explicit amendments).

## Quality gates

- Tests must pass before claiming done
- Integration proof before reporting success — type checking and unit tests verify code correctness, not feature correctness
- Don't merge what you haven't verified end-to-end
- For initiative-scale work, validate against the requirements doc — "have we satisfied every must-have?"

## Process iteration

This document is itself a living experiment. v3 will become v4 when the methodology evolves. Suggest changes when something isn't working or when a principle proves its weight. If a rule consistently gets skipped and nothing breaks, it's noise — remove it.

## What's NEW in v3 vs lean

- **Explicit pipeline shape** — stages and hand-offs are explicit, not just "do whatever feels right"
- **`/hl:red-team` skill** — turns ad-hoc Foundry refinement into a structured, fast-paced workflow (decisions per minute, not threads per session)
- **`/hl:blue-team` skill** — formalizes scope-cutting and produces a durable requirements contract per initiative
- **Requirements docs as the scope contract** — initiative-scale work gets a per-initiative requirements doc; epics reconcile to it
- **`/hl:stop` surfaces red-team gaps** explicitly — gaps that needed deferral don't get lost
- **Amendments are first-class** — scope changes are logged, not silent
