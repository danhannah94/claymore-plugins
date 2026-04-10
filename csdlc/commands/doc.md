---
description: Generate a CSDLC Step 0 design doc for a feature — problem statement, scope, stories, open questions
argument-hint: <feature name or description>
---

Create a CSDLC Step 0 design doc for: **$ARGUMENTS**

Step 0 is the design doc. It defines the WHAT and WHY before anyone writes
a line of code. The output is a doc that's ready for Step 1 (refinement).

## Procedure

1. **Understand the feature.** Read `$ARGUMENTS`. If it's terse (just a name),
   ask the user 2-3 clarifying questions FIRST before drafting:
   - What problem does this solve? (User pain, business goal, tech debt)
   - Who's affected? (End users, other devs, the system itself)
   - What does "done" look like in one sentence?

   If `$ARGUMENTS` is already descriptive, proceed — don't over-ask.

2. **Check existing context.** Before writing from scratch, check if there's
   an existing issue, design doc, or NEXT.md entry that relates. Look in
   Foundry (search_docs) and the local repo (grep). Build on what exists.

3. **Draft the design doc.** Use this structure:

   ```markdown
   # <Feature Name>

   *Status: Step 0 (draft) | Date: <absolute date> | Author: Dan + Clay*

   ## Problem Statement
   <What's broken or missing, and why it matters. 2-4 sentences max.>

   ## Goals
   - <Concrete, measurable goal>
   - <Another goal>

   ## Non-Goals (Explicit Scope Boundaries)
   - <What this is NOT trying to do>

   ## Design

   ### Approach
   <The proposed solution in prose. Architecture-level, not code-level.
   Diagrams welcome. Mention alternatives considered and why they lost.>

   ### Stories
   | # | Story | Complexity | Notes |
   |---|-------|------------|-------|
   | S1 | <user story or task> | S/M/L | <dependency or risk> |
   | S2 | ... | ... | ... |

   ### Design Decisions
   | Decision | Choice | Rationale |
   |----------|--------|-----------|
   | <question> | <answer> | <why> |

   ## Open Questions
   - [ ] <Question that needs answering before implementation>
   - [ ] <Another one>

   ## Rollback Plan
   <How to undo this if it breaks. "Not applicable" is a valid answer for
   additive-only changes, but say so explicitly.>

   ## Success Criteria
   - <How we'll know this shipped correctly>
   ```

4. **Present the draft** to the user. Don't write it to a file yet — output
   it in the conversation so they can react. Ask:
   - "Does this capture what you're thinking?"
   - "Anything I got wrong or missed?"

5. **After user feedback,** apply edits and ask where to save it:
   - Foundry (via insert/create MCP tools)
   - Local filesystem (a .md file)
   - Both

## Design principles (from PROCESS.md)

- **Resolve ambiguity, don't defer it.** If you're writing "TBD" more than
  twice, the feature isn't ready for a design doc — it needs a conversation.
- **Stories are sequenceable.** Each story should be implementable in isolation
  (or with explicit dependencies noted). If story S5 secretly requires S3+S4,
  say so.
- **Complexity estimates are gut-check, not sprint planning.** S/M/L is
  enough. Don't pretend precision you don't have.
- **Non-goals are as important as goals.** They prevent scope creep during
  implementation. "We are NOT building X" is a decision, not a deferral.
