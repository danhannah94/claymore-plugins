---
name: csdlc-doc
description: Generate a CSDLC Step 0 design doc for a feature, epic, or project. Includes bundled templates for three levels (project, epic, subsystem). Use when the user says "doc", "design doc", "Step 0", "write a design doc", or passes a feature description.
process_version: "1.0"
argument-hint: <feature name or description>
---

# CSDLC Doc — Step 0 Design Doc

Create a Step 0 design doc for: **$ARGUMENTS**

Step 0 is the design doc. It defines the WHAT and WHY before anyone writes a line of code. The output is a doc ready for refinement (`/refine`) and then story breakdown (`/breakdown`).

## Step 1: Determine the level

Design docs come in three levels (from PROCESS.md):

| Level | When to use | Template |
|-------|-------------|----------|
| **Project** | New project, full architecture | `templates/project-design.md` |
| **Epic** | New feature/epic within an existing project | `templates/epic-design.md` |
| **Subsystem** | Architectural boundary or capability | `templates/subsystem-design.md` |

**The test:** If you removed it, would multiple unrelated features break? → Subsystem. Would one specific capability disappear? → Epic. Is it a whole new project? → Project.

If the level isn't clear from `$ARGUMENTS`, ask one question: "Is this a new project, a new feature within an existing project, or defining an architectural boundary?"

## Step 2: Understand the feature

If `$ARGUMENTS` is terse (just a name), ask 2-3 clarifying questions FIRST:
- What problem does this solve? (User pain, business goal, tech debt)
- Who's affected? (End users, other devs, the system itself)
- What does "done" look like in one sentence?

If `$ARGUMENTS` is already descriptive, proceed — don't over-ask.

## Step 3: Check existing context

Before writing from scratch, check for existing docs:
- Search Foundry (`search_docs`) for related design docs
- Check the local repo for README, existing design docs, or architecture notes
- Build on what exists — don't duplicate.

## Step 4: Draft the design doc

Read the appropriate template from `templates/` in this skill's directory. Fill it in using the information gathered. Follow these principles:

- **Resolve ambiguity, don't defer it.** If you're writing "TBD" more than twice, the feature isn't ready — it needs a conversation.
- **Stories are sequenceable.** Each story should be implementable in isolation (or with explicit dependencies noted).
- **Non-goals are as important as goals.** They prevent scope creep.
- **Complexity estimates are gut-check.** S/M/L is enough.

## Step 5: Present the draft

Output the draft in the conversation so the user can react. Ask:
- "Does this capture what you're thinking?"
- "Anything I got wrong or missed?"

## Step 6: Save

After user feedback, ask where to save:
- **Foundry** (preferred) — `create_doc` at the appropriate path (`projects/{project}/design` for project-level, `projects/{project}/epics/{epic-name}` for epic-level)
- **Local filesystem** — write as a `.md` file
- **Both**

Then suggest: "Ready for `/refine` to challenge this, or `/breakdown` to extract stories?"
