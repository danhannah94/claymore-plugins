---
description: Run the CSDLC session-start ritual — load context, deliver the 5-part standup
---

Run the CSDLC session-start ritual.

Invoke the `csdlc-session-start` skill. Load context in the canonical order
(human context → process & values → current priorities → active design doc →
project workflow → long-term memory), then deliver the 5-part standup:

1. **Context loaded** — which files you read, layers skipped, why
2. **Recap** — what shipped last session (from the most recent ✅ SHIPPED block)
3. **On deck** — the top item(s) from the 🎯 Priority Stack and current 🔴 NEXT
4. **Design doc state** — refined vs open on the active design doc, if any
5. **Gaps** — anything you couldn't load, anything stale, any contradictions

Then ask **one** question: which item should we start with, or is there
something not in NEXT.md that takes priority?

Do not start the work until the standup is out and the user has confirmed
the top item (or redirected you). Follow the full skill — don't skip steps
just because `/standup` is a shortcut.
