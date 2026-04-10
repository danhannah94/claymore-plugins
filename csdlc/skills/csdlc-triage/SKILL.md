---
name: csdlc-triage
description: Triage open GitHub issues — categorize as Step 1 ready or needs design doc, generate stories for actionable ones, hand off to /execute. Use when the user says "triage", "triage issues", "what can we fix", "check the issues", or wants to batch-process open bugs and features.
process_version: "1.0"
argument-hint: "[--repo owner/repo] [--label label] [--limit N]"
---

# CSDLC Triage — Issue Pipeline

Triage open issues for: **$ARGUMENTS**

`/triage` is the intake funnel for the CSDLC pipeline. It reads open GitHub issues, decides which can enter the pipeline at Step 1 (no design doc needed) vs which need a Step 0 design doc first, generates stories for the actionable ones, and hands off to `/execute`.

## Procedure

### Phase 1: Read issues

1. **Determine the target repo.** From `--repo` flag, current directory's git remote, or ask.
2. **Fetch open issues.** `gh issue list --repo {repo} --state open --json number,title,body,labels,createdAt --limit {limit or 25}`
3. **Filter.** If `--label` was specified, filter to that label. Otherwise, process all open issues.

### Phase 2: Categorize each issue

For each issue, assess against the **Step 1 readiness criteria:**

| Criterion | Step 1 Ready | Needs Design Doc |
|-----------|:---:|:---:|
| **Problem is clear** | Yes — can state the bug/need in one sentence | No — vague, multi-faceted, or needs investigation |
| **Fix shape is obvious** | Yes — know which files to touch and roughly how | No — multiple approaches, architectural decision needed |
| **Scope is bounded** | Yes — one concern, clear boundaries | No — touches multiple systems, unclear blast radius |
| **No new architecture** | Yes — works within existing patterns | No — introduces new patterns, tables, endpoints |
| **Dependencies are clear** | Yes — standalone or known blockers | No — unclear what else needs to change |

**Categorize into three buckets:**

- **Step 1 Ready** — Clear problem, clear fix, bounded scope. Can generate a story and execute.
- **Needs Design Doc** — Too complex, too ambiguous, or architectural. Queue for Step 0.
- **Needs Investigation** — Can't categorize without more info. Needs repro, clarification, or research first.

### Phase 3: Present the triage

Show the user the categorization before acting:

```
## Triage Report: {repo} ({N} open issues)

### Ready to Execute (Step 1)
| # | Title | Complexity | Why |
|---|-------|-----------|-----|
| #98 | reopen_annotation status bug | S | One-liner state machine fix |
| #117 | content param on create_doc | S | Clear scope, additive change |

### Needs Design Doc (Step 0)
| # | Title | Why |
|---|-------|-----|
| #99 | Private docs in search | Blocked on E12 auth epic |

### Needs Investigation
| # | Title | What's unclear |
|---|-------|---------------|
| #102 | Schema cache bug | Can't reproduce, client-side |

Want me to generate stories for the Step 1 items? Or adjust the categorization?
```

### Phase 4: Generate stories (on user approval)

For each Step 1 Ready issue, generate a story with:

- **Story ID:** Auto-generate from project prefix + issue number (e.g., `FND-BUG-98`)
- **Scope:** What specifically to change, derived from the issue body
- **Acceptance criteria:** Testable criteria derived from the expected behavior in the issue
- **Target files:** Best guess from the issue context (verify by searching the codebase)
- **Boundaries:** What NOT to touch
- **Complexity:** S / M / L

Write the stories to Foundry as sections on a triage doc (e.g., `status/triage-{date}.md`) or directly as comments on the GitHub issues.

### Phase 5: Execute (on user approval)

After the user approves the stories, offer:
- `/execute` — work them with sub-agents, review each PR, wait for approval
- Cherry-pick specific issues to execute now vs later
- Adjust story scope before execution

### Phase 6: Close the loop (after execution)

For each fixed issue:

1. **Close the GitHub issue** with a comment linking the PR and summarizing the fix:
   ```
   Fixed in PR #{num}.
   
   Root cause: {one sentence}
   Fix: {one sentence}
   Tests: {what was added/verified}
   ```

2. **Update Foundry docs.** If the fix is relevant to an existing design doc:
   - Add a "Post-ship Findings" section or annotation on the relevant epic/design doc
   - If the fix revealed a pattern or gotcha, add it to the project's tech debt or lessons learned

3. **Update NEXT.md.** Add the fixes to the current session's ✅ SHIPPED block via `/retro`.

## Triage principles

- **Bias toward action.** If an issue is 80% clear, generate the story and let the review step catch gaps. Don't defer everything to a design doc.
- **One issue = one story.** Don't bundle issues into combined stories. Each fix is independently verifiable.
- **Blocked issues are still useful.** Categorizing them surfaces the dependency chain — "this is blocked on E12" is information that helps prioritize E12.
- **The triage itself is a refinement activity.** Reading issues carefully often reveals that the real problem is different from what was filed. Flag these.

## What NOT to do

- Don't auto-execute without user approval of the triage categorization
- Don't close issues without confirming the fix actually works
- Don't skip the Foundry doc update — implementation details captured now save debugging time later
- Don't triage issues you can't read the full body of (truncated issues need `gh issue view` first)
