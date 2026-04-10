---
description: Create a GitHub issue — bug, feature, tech debt, or task. Specify repo with --repo flag.
argument-hint: "<issue title> [--repo owner/repo]"
---

File an issue for: **$ARGUMENTS**

Invoke the `csdlc-issue` skill. Draft a well-structured GitHub issue
from the current context and create it with `gh issue create`.

If `--repo` is specified (e.g., `/issue auth bug --repo danhannah94/foundry`),
target that repo. Otherwise, detect from the current working directory's
git remote.
