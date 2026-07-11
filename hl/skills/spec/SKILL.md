---
name: spec
description: "Drive an epic through the Kiro-style spec pipeline: grounding → requirements (EARS, red/blue-teamed and LOCKED) → design docs with mermaid architecture (red-teamed) → tasks doc with AC-to-requirement traceability and a dependency graph in /hl:ship's input format. Use when starting any initiative-scale feature or epic; each stage gates the next so tasks inherit two rounds of hardening. Accepts an epic name or an existing requirements-doc reference to resume mid-pipeline."
argument-hint: "<epic-name | doc-ref> [--stage <n>]"
---

# /hl:spec — Kiro-Style Spec Pipeline (v1)

Requirements → design → tasks, each stage individually hardened before the next begins. By the time tasks exist, everything upstream survived a red-team. This is the initiative-scale front half of the hl pipeline; `/hl:ship` is the back half and consumes this pipeline's final artifact directly.

**Lineage:** the requirements/design/tasks artifact convention is adopted from AWS Kiro's spec mode (assessed 2026-07-10: don't migrate, steal the format). The red/blue-team gates and fresh-agent grounding are ours. First run: quoteai-next epic, session 51 (2026-07-11).

## Stage map

| # | Stage | Output | Gate | Skip when |
|---|---|---|---|---|
| 0 | Ground | Fresh repo/infra exploration report | — | You explored this area this session |
| 1 | Draft requirements | `<epic>-requirements` doc in Foundry | — | Never (this IS the pipeline) |
| 2 | Human refinement | Amended requirements | human reads it | Human says go |
| 3 | Red-team requirements | Gaps triaged + applied | findings triaged | Tiny/low-stakes epic |
| 4 | Blue-team requirements | Scope cut, priorities locked → **doc marked LOCKED** | human locks | Scope already tight (single-track epic) |
| 5 | Design doc(s) | `<epic>-design` doc(s) with mermaid | — | Trivially thin design (rare) |
| 6 | Human refinement | Amended design | human reads it | Human says go |
| 7 | Red-team design | Gaps triaged + applied | findings triaged | Low-stakes design |
| 7b | Fold-back triage | Requirements-level findings routed → Amendments + design updates | human fold-in prompts | No requirements-altitude findings |
| 9 | Tasks doc | `<epic>-tasks` doc in /hl:ship Form-A format | coverage check passes | — |

*(Stage 8 is deliberately not a stage — it's the no-blue-team-at-design rule below.)*

Stages are skippable per the low-ceremony rule — but **skipping is a call the human makes**, surfaced explicitly ("recommend skipping blue-team here because X — ok?").

## Stage 0 — Ground

Before drafting a word: spawn parallel Explore agents over the affected repos/infra to map what actually exists (current data models, dead-ends, pre-built-but-unused schema, prior decisions in decisions.md/Dev Memory). The requirements doc's factual premises come from this report, not from memory. Recall Dev Memory for prior rationale on the epic's territory (the `recall` triggers apply).

## Stage 1 — Draft requirements

One requirements doc per epic (per-STAGE docs, not per-feature — sections give the same red-team granularity without fragmenting the gates; the one exception: a substrate/platform module that outlives the epic gets its own standalone doc). Structure:

- **Header:** epic name, date, "v1 draft, pre-red-team", pointer to the umbrella/parent doc if any.
- **Global constraints** (C-series): apply to every requirement. Include the standing ones that fit: critical-path precedence, vended-account compatibility, compound-don't-duplicate, check-owner tagging, flagship-inert.
- **Numbered requirements** grouped by track/module: each has an id (`N-1`, `R-2`…), a priority (**must / should / could** — blue-team cuts against these), a user story (As/I want/So that), and **EARS acceptance criteria** (WHEN/IF … THE SYSTEM SHALL …) each tagged 🤖 (AI-mechanized check) or 🧑 (human judgment).
- **Out of scope (v1)** — explicit, named.
- **External clocks** — anything with lead time measured in days (approvals, filings, third parties). File these at epic start.
- **Amendments table** — empty at draft; every post-lock change lands here with date + why. Scope creep that's documented is feature; silent is bug.

Requirements say WHAT and WHY. Design decisions (table names, service choices, engines) do NOT live here — but *constraints* on the design do. When a design question tries to sneak in, demote it to an explicitly-flagged open question.

## Stages 2 & 6 — Human refinement (optional)

The human reads the artifact and reacts; apply amendments live (update_section + Amendments table). Cheap, high-signal. Don't gate on it if the human says go.

## Stage 3 — Red-team requirements

Invoke the `red-team` skill against the requirements doc. **The author is not the red-teamer** — use the fresh-agent fan-out with requirements-tuned lenses:

- **Factual grounding** — verify every claim about existing code/infra against the repos.
- **Completeness** — missing must-haves: identity/tenancy, lifecycle, compliance, failure modes, migration/rollout, bootstrap problems ("who assigns the first admin?").
- **Testability/EARS rigor** — untestable, unachievable ("exactly once"), contradictory, or scope-hiding criteria; wrong 🤖/🧑 tags; priority/dependency mismatches.
- **Cross-epic/enterprise** — interactions with other epics, the vend/multi-account future, the north-star roadmap.

Findings are adversarially verified (default-refute), then triaged with the human (batched AskUserQuestion rounds, recommendation-first) and applied.

## Stage 4 — Blue-team requirements (optional)

Invoke the `blue-team` skill: cut scope from the full landscape, settle must/should/could disagreements, resolve red-team items marked "defer decision." Exit: edit the doc header to **LOCKED (vN, date)**. After lock, changes go through the Amendments table only.

## Stage 5 — Design

One design doc per **module boundary** (a substrate module = standalone doc; app-local features can share). Structure: context + locked-requirements pointer; architecture with **mermaid diagrams** (component + sequence for the load-bearing flows); data model; decision series (`XX-D1`…) with alternatives-considered; **traceability — every design element cites the requirement ids it satisfies**; open questions for red-team; Amendments table. A design that can't cite a requirement for something it builds is adding scope — that's a requirements amendment, not a design freebie.

## Stage 7 — Red-team design

Fresh-agent fan-out again, design-tuned lenses:

- **Security** — authn/z boundaries, tenant isolation, injection surfaces, secrets handling.
- **Cost** — infra + token economics at realistic and 10x volume (when applicable).
- **Plan consistency** — cross-reference current and future plans: other epics' designs, the north-star roadmap, the vend/two-prod model. Does this design paint a future epic into a corner?
- **Operability & failure modes** — what breaks silently? (the lost-tag-alarm class), retries, observability, rollback.
- **Simpler alternative** — could a boring design meet every locked requirement? (adversarial check on over-engineering).
- **Requirements fidelity** — does the design actually satisfy every must? Any silent scope cuts?

## Stage 7b — Fold-back triage (requirements ↔ design reconciliation)

Design work and the design red-team both surface requirements-level material: the design docs' `Requirements amendment candidates` sections, plus any red-team finding that is really a scope/WHAT question wearing a design costume. These are folded back through ONE combined triage with the human:

1. **Route by altitude.** Each confirmed red-team finding is classified at triage: *design-level* (fix applies to the design doc) or *requirements-level* (joins the amendment candidates).
2. **Fold-in prompts.** Every requirements-level item gets an explicit human decision (AskUserQuestion, recommendation-first, batched): **fold in** → an Amendments row in the LOCKED requirements doc AND the matching design-doc update, applied in the same pass; **decline** → recorded in the design doc's red-team recap as considered-and-declined (so it doesn't resurface every round); **defer** → carried in next.md.
3. **One scope ledger, always.** The requirements doc's Amendments table is the only door scope changes enter through after lock — the lock stays meaningful because every post-lock change is visible, dated, and human-signed.

## Stage 8 — no blue-team at design (deliberate)

Scope was cut at Stage 4; design must not add or shed scope silently. If design work reveals scope pressure (a must is pricier than believed), the move is an **explicit requirements amendment** via Stage 7b's fold-back — possibly a priority downgrade — not a design-stage cut.

## Stage 9 — Tasks doc

The final artifact, in **`/hl:ship` Form-A format** so ship consumes it directly:

- `## Story Summary` table: `Story | Title | Scope | Deps | Visual | Phase`.
- `## Stories` — one section per story: scope, **acceptance criteria each citing the requirement id(s) it satisfies** (`AC: … [R-2]`), verification notes for the QA agent (how to reach/trigger the change), branch-name suggestion.
- **Dependency graph:** the Deps column drives ship's topo-sort. Encode code deps AND same-file-collision serialization (two stories touching the same shell/header get a dep even without a code dependency). Optional mermaid `graph TD` for human legibility.
- **Coverage check (mechanized, the stage gate):** cross-check that every **must** requirement id is cited by ≥1 story's AC, and every story AC cites ≥1 requirement. Run it as a checklist or a small agent pass; a must with no story = the pipeline caught scope loss — surface it, don't ship around it.

Then hand off: `/hl:ship <tasks-doc-ref>`.

## Rules

- **Artifacts live in Foundry** (Design Docs KB), not in-repo — the red-team/annotation/review tooling targets Foundry docs. Name convention: `<epic>-requirements`, `<epic>-design[-<module>]`, `<epic>-tasks`, same folderPath.
- **Author ≠ red-teamer, every time.** Both red-team stages use fresh agents grounded in the repos.
- **One scope ledger.** Priorities live in requirements; every post-lock change is an Amendments row.
- **Traceability is bidirectional at the end:** requirement → story → AC → check owner.
- **Model tiering:** finders/verifiers/builders on Opus/Sonnet; the main loop orchestrates.
- **Log the run** in decisions.md (what the gates caught, what they cost) — the pipeline is itself an experiment; evaluate at `/hl:stop`.

## Iteration log

**v1 — 2026-07-11 (session 51).** Codified from the live quoteai-next run: per-stage docs beat per-feature docs (sections give red-team granularity; one gate session instead of three); the requirements red-team lens set (grounding/completeness/testability/cross-epic) and the design lens set (security/cost/plan-consistency/operability/simpler-alternative/requirements-fidelity) were chosen here; Stage-8 no-blue-team-at-design rule adopted (scope pressure → visible requirements amendment). First full validation pending: quoteai-next through Stage 9 + ship.
