---
description: End a session — triage decisions, surface unresolved red-team gaps, write next.md for the next session. Optional project slug scopes to a project directory.
argument-hint: "[project]"
---

End session for: **$ARGUMENTS**

Run the `stop` skill. Triage decisions (update decisions.md, graduate
settled decisions to Foundry annotations), surface any unresolved red-team
gaps for the next session, and write next.md with full context for the next
session to pick up cold.

If a project slug is provided (e.g., `/hl:stop autri`), scope all file writes
to `<workspace>/<project>/`. Otherwise, write to the workspace root.

Follow the full `stop` skill — don't skip the decisions triage or red-team
gap surfacing phases.
