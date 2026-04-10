---
name: csdlc-shipit
description: Run all stories in an epic autonomously — the Hurricane Pattern. Read the dependency graph, dispatch stories to sub-agents in order, review PRs, merge, report back. Use when the user says "shipit", "run the whole epic", "full send", "hurricane", or "lightning strike this".
process_version: "1.0"
argument-hint: <epic doc path in Foundry>
---

# CSDLC Ship It — Hurricane Pattern

Ship this epic: **$ARGUMENTS**

`/shipit` is the Hurricane Pattern from PROCESS.md made into a command. The AI Lead becomes the "eye of the hurricane" — spawning batches of sub-agents, coordinating merges, resolving conflicts, and verifying builds.

## Procedure

1. **Read the epic doc.** `$ARGUMENTS` is a Foundry path to an epic design doc with stories already broken down (via `/breakdown`). If no stories exist, suggest running `/breakdown` first.

2. **Build the dependency graph.** From the stories' dependency fields:
   - Identify foundation stories (no dependencies — go first)
   - Identify parallel groups (can run simultaneously)
   - Identify serial chains (must complete in order)
   - Produce a visual execution plan for the user

3. **Confirm the plan.** Present:
   ```
   Execution Plan for {Epic}:
   
   Batch 1 (foundation): S1
   Batch 2 (parallel):   S2, S3, S4
   Batch 3 (serial):     S5 (depends on S2)
   Batch 4 (parallel):   S6, S7
   
   Total: {N} stories, {M} batches
   Estimated: {complexity-based rough guess}
   
   Full send? Or adjust the plan?
   ```

4. **Execute in batches.** For each batch:
   - Dispatch each story using the `csdlc-intern` skill workflow
   - For parallel stories, use worktree isolation (one worktree per story)
   - Wait for all stories in the batch to complete before starting the next
   - Run `/review` on each completed story
   - If a story fails review, fix and re-run before proceeding

5. **Between batches:**
   - Merge completed PRs from the previous batch
   - Verify the build on merged main
   - Resolve any merge conflicts
   - Update story status in Foundry

6. **After all batches complete:**
   - Run `/qa` on the full epic (integration-level, not per-story)
   - Run `/present` to package for human review
   - Write a summary annotation to the epic doc in Foundry

## Abort conditions

Stop execution and report to the user if:
- A foundation story fails and can't be fixed
- More than 2 stories in a batch fail
- A merge conflict can't be resolved automatically
- Build breaks after a merge and can't be fixed in 2 attempts

## What NOT to do

- Don't skip the review step between batches to "go faster." Quality gates are mandatory.
- Don't merge without a passing build.
- Don't continue past a broken foundation story — everything downstream depends on it.
- Don't run this without user confirmation of the execution plan.
