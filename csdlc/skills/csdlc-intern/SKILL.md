---
name: csdlc-intern
description: Hand off a single story to a sub-agent (intern) for execution. Reads the story from Foundry, crafts the agent prompt, spawns the intern in a worktree, writes results back to Foundry. Use when the user says "intern", "hand this off", "send it to the intern", "dispatch", "execute this story".
process_version: "1.0"
argument-hint: <story ID or Foundry path>
---

# CSDLC Intern — Sub-agent Handoff

Hand off this story to a sub-agent: **$ARGUMENTS**

This is Step 3 of the CSDLC pipeline. A refined, scoped story gets handed to an "intern" (sub-agent) for execution. The intern works in an isolated worktree, implements the story, and reports back.

## Procedure

1. **Read the story.** `$ARGUMENTS` is either:
   - A story ID (e.g., `FND-E12-S3`) → search Foundry for the story section in the epic doc
   - A Foundry path → `get_page` to read it
   - A local file path → read the file
   - Nothing → ask the user which story to dispatch

2. **Verify the story is ready.** Check that it has:
   - [ ] Clear scope
   - [ ] Acceptance criteria (specific, testable)
   - [ ] Target files/areas identified
   - [ ] Boundaries defined (what NOT to touch)
   
   If any are missing, flag them and ask: "This story is missing [X]. Want me to fill it in, or should we `/refine` it first?"

3. **Craft the agent prompt.** Use the `csdlc-craft-agent-prompt` skill to assemble the prompt. The prompt includes:
   - Role and context (story ID, repo, base branch)
   - Task context (story content + relevant design doc sections)
   - Acceptance criteria (copied directly)
   - Explicit boundaries (do-not-touch rules)
   - Verification steps (build, test commands)
   - Files to read first

4. **Confirm with the user.** Present a summary:
   ```
   Ready to dispatch:
   Story: {ID} — {title}
   Repo: {path}
   Branch: {branch-name}
   Scope: {one-line summary}
   
   Send the intern? (or review the full prompt first)
   ```

5. **Spawn the intern.** Use the Agent tool with `isolation: "worktree"`:
   - Pass the crafted prompt
   - The intern works in an isolated worktree — no contamination of the main branch
   - The intern implements, tests, and opens a PR (if configured)

6. **Report results.** When the intern completes:
   - Summarize what was done (files changed, tests passed/failed, PR link)
   - Write a result annotation to Foundry on the story doc:
     ```
     Story {ID} executed.
     Status: complete / failed
     Branch: {branch}
     PR: {url}
     Tests: {summary}
     ```
   - Suggest next steps: `/review` to verify, or dispatch the next story

## Execution modes (from PROCESS.md)

| Mode | When | How |
|------|------|-----|
| **Direct sub-agent** | Simple, well-scoped stories | Spawn intern with full prompt. Intern implements and reports back. |
| **Managed agent** | Complex stories, audit trail needed | Spawn a lightweight manager (Sonnet) that delegates to Claude Code for implementation. |

Default to direct sub-agent. Use managed agent when the story is complex enough to warrant orchestration.

## What NOT to do

- Don't execute without user confirmation (unless `/shipit` is running the batch).
- Don't modify the story doc — that's the AI Lead's job, not the dispatch skill's.
- Don't skip the prompt review step. A bad prompt produces rework.
