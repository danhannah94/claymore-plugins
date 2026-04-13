# Process — Lean Collaborative Development

This is the operating methodology for human-AI collaboration. Only load-bearing principles — no rituals, no ceremony.

## Three-file system

| File | Scope | Who writes it | Purpose |
|------|-------|---------------|---------|
| `process.md` | Workspace | Both (iterate together) | Methodology — how we work |
| `next.md` | Workspace | AI (at session end via `/stop`) | Session handoff — what to pick up next |
| `decisions.md` | Per-project | AI (during work) | Active decisions not obvious from the code |

## Core principles

**The code is the primary artifact.** If the architecture isn't legible from reading the code — module boundaries, naming, directory structure — fix the code, not the docs. Don't compensate for unclear code with more documentation.

**Foundry captures WHY. Code is WHAT.** Design docs, annotations, and refinement sessions in Foundry exist to record rationale, trade-offs, and rejected alternatives. The code already shows what exists. Don't duplicate it in prose.

**Scope before building, design while building.** Agree on boundaries (what to build, what NOT to build) before starting. Make design decisions during implementation as they surface — that's when the real constraints are visible.

**Refine when uncertain.** When the AI hits a decision with low confidence or high stakes (expensive to reverse, affects multiple modules, or depends on context only the human has), surface it explicitly and talk it through. Don't guess. This is the valuable friction — it prevents confidently building the wrong thing.

**Delegate isolated work, hand-write load-bearing work.** Sub-agents are good for self-contained modules with clear boundaries. Architecturally significant code — the parts where shape matters and decisions compound — should be written directly so the human can feel the design emerge.

**Commit messages capture WHY.** A commit message that says what changed is useless — the diff shows that. Write why the change was made, what was considered and rejected, and any non-obvious reasoning. These form a searchable, immutable decision trail.

## Decisions lifecycle

Decisions start in `decisions.md` when they're active and being validated. As they settle:

- **Active** — still being tested, might change. Lives in `decisions.md`.
- **Settled but non-obvious** — graduate to Foundry as annotations on the relevant design doc section. The reasoning matters long-term.
- **Demonstrated by the code** — prune from `decisions.md`. The code and commit messages are sufficient.

Keep `decisions.md` short — ideally 10-20 active entries. If it's growing unbounded, entries aren't graduating or getting pruned.

## Quality gates

- Tests must pass before claiming done.
- Integration proof before reporting success — type checking and unit tests verify code correctness, not feature correctness. If there's a UI, see it in a browser.
- Don't merge what you haven't verified end-to-end.

## Session lifecycle

**`/start`** — reads `next.md`, loads context, bootstraps the session. Always begin here.

**During work** — update `decisions.md` as real decisions get made. Use Foundry for refinement on high-level architecture and design rationale. Surface uncertain decisions to the human.

**`/stop`** — writes `next.md` for the next session. Captures what was done, what's unfinished, and what context the next session needs to start informed.

## Foundry — the knowledge layer

Foundry is the project knowledge base. Design docs live there. Use Foundry MCP tools (`mcp__foundry__*`) for:
- Reading design docs and project context
- Creating annotations for design decisions and rationale
- Refinement — working through scope, architecture, and trade-offs on doc sections
- `/reply` — responding to human feedback on docs
- `/update` — locking in agreed decisions and updating doc prose

## Process iteration

This document is a living experiment. Both human and AI should suggest changes when something isn't working or when a principle proves its weight. If a rule consistently gets skipped and nothing breaks, it's noise — remove it. If a failure keeps recurring, there's a missing principle — add it.
