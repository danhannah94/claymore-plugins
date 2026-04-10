---
name: csdlc-process
description: The CSDLC methodology reference — pipeline steps, roles, refinement practices, rituals, context management. Use whenever you need to understand how CSDLC works, when a user asks about the process, when you're deciding which pipeline step applies, or when crafting agent prompts. Also triggers when terms like "refinement", "design doc", "Step 0", "Lightning Strike", "managed agent", "Hurricane Pattern" come up without clear context.
---

# CSDLC Process Reference

This skill provides the full Claymore Software Development Lifecycle methodology.
Read `PROCESS.md` (sibling file) for the complete reference. Below is a quick
orientation so you know what's in there and when to consult which section.

## When to read PROCESS.md

- Starting a new project or epic → read **Step 0: Design Doc** + **Refinement**
- Breaking work into stories → read **Step 1: Story Breakdown**
- About to spawn sub-agents → read **Step 2: Agent Prompt Crafting** + **Step 3: Sub-agent Execution**
- Reviewing agent output → read **Step 4: AI Lead Review** + **Step 5: QA**
- Session start/end → read **Rituals** (Standup, Retrospective)
- Context feels bloated or stale → read **Context Management**
- Naming a branch, PR, or ticket → read **Naming Conventions**
- Someone asks "how does CSDLC work?" → read the whole thing, it's 384 lines

## Quick reference (do NOT substitute for reading the full doc)

**Pipeline:** Refinement (continuous) → Step 0 (Design Doc) → Step 1 (Story Breakdown) → Step 2 (Agent Prompt Crafting) → Step 3 (Execution) → Step 4 (AI Lead Review) → Step 5 (QA) → Step 6 (Human Review → Ship)

**Roles:** Human = Technical Product Owner + QA Director. AI Lead = Architect + Scrum Master + Tech Lead. Sub-agents = disposable dev team.

**Entry points:** New project/epic → Step 0. Bug fix / small task → Step 1.

**Core principle:** Refinement is the practice, not a step. Resolve ambiguity before code. 30 minutes of Step 0 saves hours of rework.
