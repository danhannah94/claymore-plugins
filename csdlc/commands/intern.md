---
description: Hand off a story to a sub-agent (intern) for execution
argument-hint: "<story ID or path>"
---

Send this to the intern: **$ARGUMENTS**

Invoke the `csdlc-intern` skill. Read the story, craft an agent prompt,
spawn a sub-agent in an isolated worktree, and write results back to
Foundry when complete.
