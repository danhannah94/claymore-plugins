---
name: csdlc-qa
description: Run QA against acceptance criteria for a completed story (CSDLC Step 5). Verify each criterion, test edge cases, provide evidence. Separate from implementation — dedicated QA step. Use when the user says "qa", "test this", "verify", "does it work", or after /review passes.
process_version: "1.0"
argument-hint: <story ID, PR URL, or branch name>
---

# CSDLC QA — Step 5

QA this deliverable: **$ARGUMENTS**

Step 5 is a **dedicated QA step, separate from implementation.** Don't combine with the review (Step 4). The review checks "did the intern follow the spec?" QA checks "does it actually work?"

## Procedure

1. **Identify the deliverable.** Same as `/review` — PR URL, branch, or story ID.

2. **Load acceptance criteria.** From the story doc (Foundry or local).

3. **Verify each criterion independently.** For every acceptance criterion:
   - **How to verify:** What command, test, or action proves this criterion is met?
   - **Result:** Pass or fail, with specific evidence (test output, screenshot, log line)
   - **Edge cases:** Test at least one edge case per criterion if applicable

4. **Test beyond the criteria.** Look for:
   - **Error states:** What happens when inputs are invalid?
   - **Boundary conditions:** Empty lists, null values, max-length strings
   - **Regression:** Did anything else break? Run the full test suite.
   - **Integration:** Does it work with the rest of the system, not just in isolation?

5. **Produce the QA report.**

   ```
   ## QA Report: {Story ID}

   ### Verdict: PASS / FAIL

   ### Acceptance Criteria Verification
   | # | Criterion | Method | Result | Evidence |
   |---|-----------|--------|--------|----------|
   | 1 | {criterion} | {how tested} | PASS/FAIL | {output/screenshot} |
   | 2 | ... | ... | ... | ... |

   ### Edge Cases Tested
   | Scenario | Expected | Actual | Result |
   |----------|----------|--------|--------|
   | {edge case} | {expected behavior} | {actual behavior} | PASS/FAIL |

   ### Regression Check
   - Test suite: {N} passed, {M} failed (was {prev} before)
   - Build: pass/fail

   ### Issues Found
   - {issue description} — severity: {high/medium/low}

   ### Human Judgment Needed
   - {anything that needs human eyes — UX, aesthetics, "does this feel right?"}
   ```

6. **Write to Foundry.** Post the QA report as an annotation on the story doc.

7. **If FAIL:** List specific issues. Don't fix them in this step — QA is separate from implementation. Either:
   - Re-dispatch to the intern with fix instructions (`/intern`)
   - Flag for human decision

8. **If PASS:** The deliverable is ready for Step 6 (Human Review). Suggest `/present`.

## What NOT to do

- **Don't combine QA with implementation.** This is a separate step for a reason.
- **Don't skip edge cases** because the happy path works. The edge cases are where bugs live.
- **Don't mark PASS without evidence.** "Looks good" is not QA. Concrete evidence for every criterion.
- **Flag anything requiring human judgment.** UX, aesthetics, and "does this feel right" are human calls — surface them explicitly.
