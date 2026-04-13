---
name: lean-process
description: "This skill provides the lean collaborative development methodology. Use when the AI needs to understand the three-file system (process.md, next.md, decisions.md), session lifecycle (/start, /stop), decisions lifecycle (active → settled → demonstrated), Foundry conventions, quality gates, or when asked 'how does this process work?'"
---

# Lean Process — Reference Skill

Read `PROCESS.md` (sibling file in this skill directory) for the complete methodology.

## Quick reference

| Section | What it covers |
|---------|---------------|
| **Three-file system** | Roles of process.md, next.md, decisions.md |
| **Core principles** | Code is primary artifact, Foundry captures WHY, scope before building, refine when uncertain |
| **Decisions lifecycle** | Active → settled (Foundry annotation) → demonstrated (prune) |
| **Quality gates** | Tests pass, integration proof, don't merge unverified |
| **Session lifecycle** | /start (bootstrap) → work → /stop (handoff) |
| **Foundry** | Knowledge layer — design docs, annotations, refinement, /reply, /update |
| **Process iteration** | Living document — suggest changes when something isn't working |

## Key commands

- `/start` — bootstrap session (load next.md, process.md, orient)
- `/stop` — end session (triage decisions, write next.md)
- `/reply` — respond to Foundry annotations on a doc
- `/update` — lock in agreed decisions, update doc sections, resolve threads
