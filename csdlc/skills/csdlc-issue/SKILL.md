---
name: csdlc-issue
description: Create a GitHub issue from the current context — bug reports, feature requests, tech debt items. Use when the user says "file an issue", "open a bug", "track this", "create an issue", or when a problem is discovered during a session that needs tracking.
process_version: "1.0"
argument-hint: <issue title or description> [--repo owner/repo]
---

# CSDLC Issue — GitHub Issue Creation

File an issue for: **$ARGUMENTS**

Create a well-structured GitHub issue from the current conversation context. Used for bug reports, feature requests, tech debt tracking, or anything that needs to be tracked in the project's GitHub repo.

## Procedure

1. **Determine the target repo.** Check in this order:
   - `--repo` flag in `$ARGUMENTS` (e.g., `--repo danhannah94/foundry`) — always wins
   - Current working directory's git remote
   - Active project from NEXT.md
   - Ask the user if still ambiguous

2. **Determine the issue type.** From context or ask:
   - **Bug** — something is broken
   - **Feature** — something new to build
   - **Tech debt** — something that works but shouldn't stay this way
   - **Task** — a discrete unit of work

3. **Draft the issue.** Format based on type:

   **Bug:**
   ```markdown
   ## Description
   {What's broken, in one paragraph}

   ## Repro Steps
   1. {Step to reproduce}
   2. {Next step}

   ## Expected vs Actual
   - **Expected:** {what should happen}
   - **Actual:** {what happens instead}

   ## Context
   - Discovered during: {session context}
   - Related to: {PR, epic, or story if applicable}
   - Severity: {high/medium/low}
   ```

   **Feature / Tech Debt / Task:**
   ```markdown
   ## Description
   {What and why, in one paragraph}

   ## Context
   - Related to: {epic, design doc, or story}
   - Priority: {high/medium/low}
   - Blocked by: {dependencies, if any}
   ```

4. **Present the draft.** Show the user the title + body before creating.

5. **Create the issue.** Use `gh issue create --title "..." --body "..."` with appropriate labels if the repo has them.

6. **Report back.** Show the issue URL and number. If this is related to an active epic, suggest adding it to the NEXT.md tech debt section.

## Labels (suggest, don't require)

| Type | Suggested label |
|------|----------------|
| Bug | `bug` |
| Feature | `enhancement` |
| Tech debt | `tech-debt` |
| Task | — |

## What NOT to do

- Don't create duplicate issues — search existing issues first (`gh issue list --search "..."`)
- Don't file issues for things that should just be fixed right now
- Don't add excessive labels or metadata — keep it lightweight
