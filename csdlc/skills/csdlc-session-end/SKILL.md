---
name: csdlc-session-end
description: Write the session handoff into NEXT.md at the end of a CSDLC working session. Use when the user says "wrap up", "session end", "update NEXT.md", "close out", "good stopping point", "let's pick this up tomorrow", or when a session is clearly ending. Trigger proactively — a missing handoff costs real time next session.
---

# CSDLC Session End

Wrap the session by updating `NEXT.md` (in Foundry: `status/next.md`). NEXT.md is a **running journal that accumulates** — it's not a bootloader that gets rewritten from scratch. The most recent session's work lands on top; prior SHIPPED sections stay in place as historical record.

## Before you write

1. `get_page('status/next.md')` — read the current full contents. You need every prior section because you'll be writing the whole doc back (see safe-write below).
2. Skim your own session transcript for: what shipped, what got filed, decisions locked in, gotchas hit, tech debt added, priority shifts.
3. If the current focus changed this session, note it — the Priority Stack and 🔴 NEXT section both need updating.

## NEXT.md structure (the canonical format)

Match what's already there. The format evolved from openclaw and looks like this:

```
# NEXT.md — What We're Working On

*Updated: <absolute date, e.g. April 9, 2026> (<short session tag>)*

## 🎯 Priority Stack

1. <emoji> **<Project>** — <one-line status or deadline>
2. ...

---

## 🔴 NEXT — <current focus name>

<One-paragraph "why this is the top priority right now">

### <Ticket ID or short name> — <one-line title> (<severity emoji>)

<Repro / context / fix shape in prose, not bullets unless listing files>

### <Next ticket> ...

---

## ✅ SHIPPED — <date> <session tag>

<Session arc in a few sentences: what we tried, what pivoted, what shipped>

### Issues filed
- **#NNN** — <title> (<severity>)

### PR shipped
- **PR #NNN** — <title> — <status>

### Process lessons
- <One-line lesson with concrete trigger, not a platitude>

---

## ✅ SHIPPED — <prior session, PRESERVE as-is>
...

---

## 🟡 ON DECK (No Rush)

### <Project> — <status>
...

---

## 📋 Known Tech Debt

### <Project>
- [ ] <item>
- [x] ~~<completed>~~ — <how it was resolved>

### Process
- [ ] <process-level debt>
```

## Writing rules

- **Dates are absolute, never relative.** "Yesterday" loses meaning in 48 hours. Use `April 9, 2026`.
- **Emoji per project/section is load-bearing** — it's how you scan the doc at a glance. Follow the existing emoji scheme: 🎯 Priority Stack, 🔴 NEXT, ✅ SHIPPED, 🟡 ON DECK, 📋 Tech Debt, and per-project emoji in the Priority Stack.
- **Preserve prior SHIPPED sections verbatim.** Do not summarize, condense, or delete them. The accumulating log is the point.
- **Process lessons are concrete, not platitudes.** "Don't refine on top of broken tooling" ✅. "Be careful" ❌. Tie each lesson to the specific thing that triggered it this session.
- **Tech Debt uses checkboxes.** `- [ ]` for open, `- [x] ~~strikethrough~~` for completed, so you can see progress without losing history.
- **Voice:** terse, dry, honest. Flag risks, mark slippage ("don't let this slip — 11 days to go"). Matches the rest of the workspace.

## How to write it safely (Foundry)

NEXT.md is a single Foundry page. Surgical section-by-section edits work, but the rewrite pattern is preferred for NEXT.md because multiple sections change per update and one atomic rewrite is cleaner than N sequential patches:

1. Compose the full new body as a single markdown string, including every prior SHIPPED section unchanged.
2. `delete_doc('status/next.md')`.
3. `create_doc('status/next.md', template='blank')`.
4. `update_section(path='status/next.md', heading_path='# NEXT.md — What We're Working On', content=<full body WITHOUT the H1 — Foundry auto-parses H2+ into children>)`.
5. `get_page('status/next.md')` and verify every section is at the right level with no `[2]` duplicates.

If any step fails, don't retry surgically — re-run the rewrite from step 2.

## Edge cases

- **Running out of context:** Minimum viable update — a new ✅ SHIPPED block (one paragraph) and a refreshed 🔴 NEXT. Leave Priority Stack and Tech Debt untouched. Flag to user that it's abbreviated.
- **No prior NEXT.md:** First session — write the inaugural version with all sections but only one ✅ SHIPPED block.
- **Mid-task, no clean handoff point:** Write a **Things I'm unsure about** subsection inside 🔴 NEXT. Honesty beats fake closure.
- **`delete_doc` unavailable:** Fall back to a single `update_section` on the H1 with the full body (no embedded H1 in content). Verify with `get_page` twice.
- **Priority changed mid-session:** Move the new top item to position 1 in Priority Stack, explain why in the ✅ SHIPPED block for this session so the reordering has a paper trail.
