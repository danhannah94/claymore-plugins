---
name: update
description: "Lock in agreed decisions from Foundry annotation threads. Updates doc sections to reflect agreed changes, resolves completed threads, and graduates settled decisions to decisions.md. Use after /reply when threads have reached consensus, or when the human says to update the doc, lock in decisions, or apply feedback."
argument-hint: "[doc path]"
---

# /update — Lock In Decisions

Read annotation threads where agreement has been reached, update the document to reflect those decisions, and resolve the threads. This is the high-stakes counterpart to `/reply` — it edits the doc and closes discussions.

## Steps

1. **Determine the target doc.** If `$ARGUMENTS` is provided, use it as the Foundry doc path. Otherwise, check `next.md` for the active design doc. If neither, ask the human.

2. **Fetch the doc.** Call `mcp__foundry__get_page` for the full document structure and current content.

3. **List annotation threads.** Call `mcp__foundry__list_annotations` with `status: "submitted"` and `status: "replied"`. For each top-level annotation, call `mcp__foundry__get_annotation` to load the full thread.

4. **Identify threads with clear agreement.** A thread has agreement when:
   - Both human and AI have weighed in (at least one reply exists)
   - The most recent messages indicate acceptance, agreement, or a clear resolution
   - There is no open question or unresolved pushback

   **If uncertain whether agreement exists, skip the thread.** Note it in your summary as needing more discussion. Err on the side of not updating — false agreement is worse than delayed agreement.

5. **Update doc sections.** For each agreed thread:
   - Read the current section via `mcp__foundry__get_section`
   - Determine what change the thread agreed on
   - Write the updated content
   - Call `mcp__foundry__update_section` — target the **deepest heading possible** (leaf edits, not H1 rewrites) to preserve annotations on sibling sections
   - Verify the update by calling `mcp__foundry__get_section` after writing

6. **Resolve completed threads.** For each thread whose decision is now reflected in the doc, call `mcp__foundry__resolve_annotation` on the top-level annotation. This marks the discussion as done.

7. **Graduate decisions.** For each resolved thread, check if the decision belongs in `decisions.md`:
   - If it's active and still being validated elsewhere: add or update the entry in `decisions.md`
   - If it's now settled and non-obvious: it lives in the Foundry annotation (which was just resolved with rationale). No `decisions.md` entry needed unless it's cross-cutting.
   - If it's demonstrated by the code: no entry needed anywhere

8. **Summarize.** Output:
   - Which sections were updated and what changed (one line each)
   - Which annotation threads were resolved
   - Which threads were skipped (no clear agreement) — suggest running `/reply` on these
   - Any `decisions.md` changes made

## Rules

- **Leaf edits only.** Use `update_section` on the most specific heading possible. Updating an H1 replaces everything under it, including sections with their own annotations. Target H2, H3, or deeper.
- **Verify after writing.** Always call `get_section` after `update_section` to confirm the change landed correctly.
- **Conservative agreement detection.** If you're not sure both parties agree, don't update. The human can always explicitly say "update this one."
- **Don't resolve without updating.** If a thread has agreement but you can't figure out what doc change to make, skip it and flag it in the summary.
- If zero threads have agreement, say so and suggest running `/reply` first.
