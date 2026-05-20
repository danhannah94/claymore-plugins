---
description: Bootstrap a new session — load methodology, read next.md for context, orient. Optional project slug scopes to a project directory.
argument-hint: "[project]"
---

Bootstrap session for: **$ARGUMENTS**

Run the `start` skill. Load `methodology.md` for v3 principles, read `next.md`
for session context, check `decisions.md` for active decisions, check Foundry
for unresolved annotations, and give a 3-5 line orientation before starting work.

If a project slug is provided (e.g., `/hl:start autri`), scope all file reads
to `<workspace>/<project>/`. Otherwise, use the workspace root.

Do not start work until the orientation is delivered and the user confirms.
