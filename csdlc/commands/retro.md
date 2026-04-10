---
description: End a CSDLC session — write handoff to Foundry NEXT.md
---

Wrap the current session by running the CSDLC retro.

Invoke the `csdlc-retro` skill. Steps:

1. Fetch the current NEXT.md from Foundry — you need the full contents
   because NEXT.md is an accumulating journal, not a scratchpad.
2. Skim your session transcript for: what shipped, what got filed,
   decisions locked in, gotchas hit, tech debt added, priority shifts.
3. Compose the new NEXT.md body. Match the existing format exactly:
   - Priority Stack at the top
   - 🔴 NEXT section for the current focus
   - A new ✅ SHIPPED block for THIS session at the top of the SHIPPED stack
   - **Preserve every prior SHIPPED section verbatim**
   - ON DECK and Tech Debt sections updated if relevant
4. Use absolute dates (never "yesterday" / "tomorrow").
5. Write back via safe-write pattern (delete_doc → create_doc →
   update_section → verify with get_page).
6. Report: one-paragraph summary + tech debt + process lessons.

Then stop. Retro doesn't start new work.
