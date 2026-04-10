---
name: csdlc-session-start
description: Run the CSDLC session-start ritual — load context in the canonical order, then deliver the 5-part standup. Use at the start of any CSDLC working session, or whenever the user says "catch up", "what are we working on", "bootload", "read NEXT.md", "standup". This skill should fire at every session start even if the user doesn't name it — orientation before action is a core CSDLC practice (PROCESS.md > Rituals > Standup).
---

# CSDLC Session Start

A new session is starting on a CSDLC project. Load context in the canonical order defined in PROCESS.md, then deliver the 5-part standup. Do not start the work until the standup is out and the user has confirmed the top item (or redirected you).

## Context loading order

Load in this order. Skip any layer that doesn't exist in this workspace — don't fabricate.

1. **Core values** (MANDATORY) — Read `CORE_VALUES.md` from the `csdlc-process` skill. These are the principles that guide how you work. Do not skip this — it's the foundation everything else builds on.
2. **Process orientation** — Read the `csdlc-process` SKILL.md (the wrapper, NOT the full PROCESS.md). This gives you the pipeline overview, roles, and when to consult the full methodology. Only read the full PROCESS.md if you need specific step details during the session.
3. **Current priorities** — `status/next.md` from Foundry. The running journal: Priority Stack, active NEXT section, recent SHIPPED, ON DECK, Tech Debt.
4. **Active design doc** — NEXT.md should name the current focus and point at a design doc. Fetch it from Foundry.
5. **Project workflow doc** — if the current focus is in a specific repo, load its `WORKFLOW.md`. Lazy-load: only when the work requires it.
6. **Long-term memory** — curated memory files only if they exist and you're in a main session (not a shared context).

NEXT.md and design docs live in Foundry. Use `get_page` from the Foundry MCP connector. If the connector isn't authenticated, surface that first — don't guess at state. Core values and process orientation come from the plugin's bundled files.

## The standup (deliver unprompted)

Before doing any work, report these five things. This is PROCESS.md > Rituals > Standup — it's a calibration ritual, not a greeting.

1. **Context loaded** — which files you read, with rough token cost if notable. Name the layers above you skipped and why.
2. **Recap** — what shipped last session, pulled from the most recent `✅ SHIPPED` block in NEXT.md.
3. **On deck** — the top item(s) from the 🎯 Priority Stack and the current 🔴 NEXT section.
4. **Design doc state** — what's refined vs still open on the active design doc, if there is one.
5. **Gaps** — anything referenced that you couldn't load, anything that looks stale, any contradictions between docs.

Then ask **one** question: which item does the user want to start with, or is there something not in NEXT.md that takes priority? Don't wait on the standup itself — deliver it, then ask.

## After the standup

- If the user confirms the top item, start on it. Don't ask permission for each next step.
- If the user redirects, update NEXT.md's Priority Stack at session end to reflect the shift.
- Do not execute anything from the "First actions" or "On deck" list before confirmation — those are proposals, not a script.

## Edge cases

- **No NEXT.md or equivalent:** Surface it, offer a cold-start conversation. Don't fabricate state.
- **Referenced docs unfetchable:** Name them explicitly in the Gaps section of the standup.
- **NEXT.md looks corrupted (duplicate sections, `[2]` suffixes):** Surface the contradiction and re-read from the source design doc it links.
- **Stale context:** If NEXT.md's top-of-file date is more than a few days old, flag it in Gaps — something may have shipped out-of-band.
- **User rushes you past the standup:** Deliver the standup anyway, just terse. Skipping it trades two minutes now for an hour of rediscovery later.
