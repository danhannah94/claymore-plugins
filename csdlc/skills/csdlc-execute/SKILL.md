---
name: csdlc-execute
description: Execute stories with sub-agents — work the stories, review each PR, wait for human approval before merging. Bundles Steps 3 (execution), 4 (review), and optionally 5 (QA). The supervised execution mode. Use when the user says "execute", "work the stories", "run these", "go build this", or after /breakdown produces stories.
process_version: "1.0"
argument-hint: <epic doc path, or "next" for current stories>
---

# CSDLC Execute — Supervised Story Execution

Execute the stories from: **$ARGUMENTS**

`/execute` is the supervised execution mode — Steps 3, 4, and optionally 5 bundled together. Work the stories with sub-agents, review each PR, and wait for human approval before merging or moving on.

## How it differs from other execution commands

| Command | Scope | Human gate | Merges? |
|---------|-------|-----------|---------|
| `/execute` | Multiple stories | **Yes** — waits for approval per PR | Only when told to |
| `/shipit` | Full epic | Minimal — autonomous hurricane | Yes, auto-merges between batches |

`/execute` is for when you want to stay in the loop. `/shipit` is for full send.

## Procedure

1. **Read the stories.** From the epic doc in Foundry or `$ARGUMENTS`:
   - Identify all stories with their dependencies
   - Build the execution order (serial vs parallel)

2. **Present the plan.** Show the user:
   ```
   Stories to execute:
   1. FND-E12-S1: Schema + DAOs (foundation, goes first)
   2. FND-E12-S2: GitHub OAuth (depends on S1)
   3. FND-E12-S3: .well-known endpoints (parallel with S2)
   
   I'll work each story, open a PR, and wait for your go-ahead
   before merging. Ready?
   ```

3. **Execute each story.** For each story in order:
   a. Craft the agent prompt (using `csdlc-craft-agent-prompt` skill)
   b. Spawn a sub-agent in an isolated worktree
   c. When the intern completes, run the review checklist (Step 4):
      - Scope match
      - Tests pass
      - Acceptance criteria met
      - No boundary violations
   d. If QA is applicable (non-trivial changes), run QA checks (Step 5)
   e. Write a result annotation to Foundry
   f. **Present the PR to the user and WAIT:**
      ```
      PR ready: FND-E12-S1 — Schema + DAOs
      Branch: fnd-e12-s1-schema-daos
      PR: #15
      Review: PASS (all criteria met)
      Tests: 12 new, 188 existing — all green
      
      Approve and merge? Or want to review first?
      ```

4. **On approval:** Merge the PR, move to the next story.
   **On rejection:** Note the feedback, fix or re-dispatch, present again.

5. **Between stories:** If the next story depends on a just-merged PR, pull main and verify the build before continuing.

6. **When all stories complete:** Run `/present` to package the full set for final review.

## Parallel execution

When stories can run in parallel (no dependencies between them):
- Spawn multiple sub-agents in separate worktrees simultaneously
- Present all PRs together when the batch completes
- User can approve/reject individually

## What NOT to do

- **Don't auto-merge.** That's `/shipit` territory. `/execute` always waits.
- **Don't skip review between stories.** Every PR gets the Step 4 checklist.
- **Don't continue past a failed story** without user input — they might want to adjust the approach.
