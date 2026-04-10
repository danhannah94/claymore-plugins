---
description: Run the CSDLC session-end ritual — write the session handoff into NEXT.md
---

Wrap the current session by running the CSDLC session-end ritual.

Invoke the `csdlc-session-end` skill. Steps:

1. Fetch the current `status/next.md` from Foundry — you need the full
   contents because NEXT.md is an accumulating journal, not a scratchpad.
2. Skim your own session transcript for: what shipped, what got filed,
   decisions locked in, gotchas hit, tech debt added, priority shifts.
3. Compose the new NEXT.md body. Match the existing format exactly:
   - Priority Stack at the top
   - 🔴 NEXT section for the current focus
   - A new ✅ SHIPPED block for THIS session at the top of the SHIPPED stack
   - **Preserve every prior SHIPPED section verbatim** — the accumulation is the point
   - ON DECK and Tech Debt sections updated if relevant
4. Use absolute dates (never "yesterday" / "tomorrow").
5. Write it back via the safe-write pattern (delete_doc → create_doc →
   update_section → verify with get_page).
6. Report to the user: a one-paragraph summary of what you logged, plus
   any tech debt or process lessons you captured.

Then stop. Session-end doesn't start new work.
