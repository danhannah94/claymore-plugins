---
description: Make scope cuts from a red-team menu and produce the requirements doc. Defends time and resources by deciding what's actually in scope. Run after /hl:red-team.
argument-hint: "<initiative-name>"
---

Blue-team scope cuts for: **$ARGUMENTS**

Run the `blue-team` skill. Five phases: Aggregate (pull the full menu from prior red-team output + epic drafts) → Initial sort (AI proposes must/should/nice/out-of-scope) → Triage (interactive rounds, recommendation-first) → Write requirements doc → Recap.

If no $ARGUMENTS provided, ask the user which initiative to blue-team.

Don't ask permission to share the menu — share it directly with the initial sort. Honest tradeoff tables when scope-cutting is non-obvious.
