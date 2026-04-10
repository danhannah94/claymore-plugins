---
description: Triage open GitHub issues — categorize, generate stories, hand off to /execute
argument-hint: "[--repo owner/repo] [--label label] [--limit N]"
---

Triage open issues: **$ARGUMENTS**

Invoke the `csdlc-triage` skill. Read open GitHub issues, categorize as
Step 1 ready (execute now) vs needs design doc (Step 0) vs needs investigation.
Generate stories for actionable issues, then hand off to `/execute`.

After fixes ship, close issues with implementation details and update
relevant Foundry design docs.
