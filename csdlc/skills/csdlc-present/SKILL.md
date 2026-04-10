---
name: csdlc-present
description: Package completed work for human review (CSDLC Step 6). Summarize what shipped, link PRs, flag judgment calls. Optimized for async review. Use when the user says "present", "show me what shipped", "package this", "ready for review", or after /qa passes.
process_version: "1.0"
argument-hint: <story ID, epic name, or "all" for current session>
---

# CSDLC Present — Step 6 Human Review Package

Package this for review: **$ARGUMENTS**

Step 6 is where the human reviews with fresh eyes. This skill packages the work into a format optimized for async review — the user may not be at the same machine where the work was done. The presentation needs to be scannable, complete, and honest about what needs human judgment.

## Procedure

1. **Gather the deliverables.** Collect for the specified scope:
   - Stories completed this session (or the specified story/epic)
   - PR links for each story
   - Review reports from `/review` (Step 4)
   - QA reports from `/qa` (Step 5)
   - Any issues filed during implementation

2. **Produce the presentation.**

   ```
   ## Ready for Review: {Epic or Story ID}

   ### Summary
   {2-3 sentences: what was built, why, and the overall result}

   ### What Shipped
   | Story | Title | PR | Tests | Status |
   |-------|-------|-----|-------|--------|
   | {ID} | {title} | [#{num}]({url}) | {pass/fail} | Ready |

   ### Changes at a Glance
   - {N} files changed across {M} PRs
   - {X} new tests added
   - {Y} lines added, {Z} removed

   ### Needs Your Judgment
   These items passed QA but need human eyes:
   - {UX decision or aesthetic call}
   - {Edge case where "correct" is debatable}
   - {Anything that feels off but isn't technically wrong}

   ### Known Limitations
   - {What this doesn't handle yet}
   - {Tech debt introduced}

   ### Process Notes
   - {Anything unusual about how this was built}
   - {Lessons learned during implementation}
   ```

3. **Write to Foundry.** Post the presentation as a review on the epic doc in Foundry using `submit_review`. This makes it visible to anyone reading the project docs.

4. **Report to the user.** Surface the presentation in the conversation. End with:
   - "Ready for your review. Anything you want to dig into?"

## What makes a good presentation

- **Scannable on any device.** Tables, not walls of text. Links, not inline diffs.
- **Honest about gaps.** If something is hacky or incomplete, say so. The human's fresh eyes catch what the AI Lead's tired eyes miss.
- **"Needs Your Judgment" is the key section.** This is where the human's value is highest — the things no AI can decide. Don't pad it with things you can decide yourself, but don't skip it either.
- **PR links are mandatory.** The human needs to click through to the actual code if they want to dig in.
