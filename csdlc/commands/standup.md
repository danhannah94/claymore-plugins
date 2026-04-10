---
description: Start a CSDLC session — load context, deliver standup. Optional persona arg (e.g., /standup clay)
argument-hint: "[persona name]"
---

Run the CSDLC session-start ritual.

Invoke the `csdlc-standup` skill. If a persona was specified (`$ARGUMENTS`),
load that persona and adopt its voice for the session.

Load context in the canonical order (core values → process → NEXT.md from
Foundry → active design doc → project workflow → memory), then deliver the
5-part standup:

1. **Context loaded** — which files you read, layers skipped, why
2. **Recap** — what shipped last session (from the most recent ✅ SHIPPED block)
3. **On deck** — top items from 🎯 Priority Stack and current 🔴 NEXT
4. **Design doc state** — refined vs open on the active design doc
5. **Gaps** — anything you couldn't load, stale content, contradictions

Then ask **one** question: which item should we start with?

Do not start work until the standup is delivered and the user confirms.
Follow the full `csdlc-standup` skill — don't skip steps.
