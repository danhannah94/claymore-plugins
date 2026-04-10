---
name: csdlc-review
description: Structured review of sub-agent output (CSDLC Step 4). Run the AI Lead review checklist before presenting work to the human. Use when the user says "review", "check this", "did the intern do it right", or after a sub-agent completes a story.
process_version: "1.0"
argument-hint: <PR URL, branch name, or story ID>
---

# CSDLC Review — Step 4 AI Lead Review

Review the output from: **$ARGUMENTS**

Before presenting work to the human, the AI Lead reviews it against the story's acceptance criteria and quality standards. This is the gate between "the intern did something" and "the human should look at this."

## Procedure

1. **Identify what to review.** `$ARGUMENTS` is either:
   - A PR URL → fetch the PR diff and details via `gh`
   - A branch name → `git diff main..{branch}`
   - A story ID → find the story in Foundry, then check for associated branches/PRs
   - Nothing → ask the user what to review

2. **Load the story's acceptance criteria.** Find the original story (from Foundry or local doc) and pull the acceptance criteria checklist.

3. **Run the review checklist.** For each item, mark pass/fail with evidence:

   ### Scope match
   - [ ] Output matches ticket scope — no extras, no missing pieces
   - [ ] No files modified outside the stated target files/areas
   - [ ] Boundaries respected — nothing in the "do not modify" list was touched

   ### Quality
   - [ ] Code is clean and follows project conventions
   - [ ] No obvious bugs, edge cases, or error handling gaps
   - [ ] Naming is consistent with the codebase
   - [ ] No unnecessary changes (reformatting, import reordering, etc.)

   ### Verification
   - [ ] Build passes
   - [ ] Tests pass (existing + any new ones)
   - [ ] No regressions introduced
   - [ ] New functionality has test coverage (if applicable)

   ### Acceptance criteria
   - [ ] Each acceptance criterion from the story is met (check individually)

4. **Produce the review report.**

   ```
   ## Review Report: {Story ID}

   ### Verdict: PASS / FAIL / PASS WITH NOTES

   ### Acceptance Criteria
   - [x] {criterion 1} — verified by {evidence}
   - [x] {criterion 2} — verified by {evidence}
   - [ ] {criterion 3} — FAILED: {what's wrong}

   ### Scope Check
   - Files modified: {list}
   - Out-of-scope changes: {none / list}
   - Boundary violations: {none / list}

   ### Quality Notes
   - {Any concerns, suggestions, or things the human should look at}

   ### Test Results
   - Build: pass/fail
   - Tests: {N} passed, {M} failed
   - New tests added: {count}

   ### Recommendation
   {Ready for human review / Needs fixes first / Needs discussion}
   ```

5. **Write result to Foundry.** Post the review report as an annotation on the story doc in Foundry using `create_annotation`.

6. **If FAIL:** Identify specific fixes needed. Offer to either fix them directly or re-dispatch to the intern with the fix instructions.

7. **If PASS:** Suggest `/present` to package for human review, or proceed to the next story.

## What NOT to do

- Don't rubber-stamp. Actually check each acceptance criterion.
- Don't fix issues silently — surface them. The human needs to know what was caught.
- Don't block on style nits. Focus on correctness, scope, and acceptance criteria.
