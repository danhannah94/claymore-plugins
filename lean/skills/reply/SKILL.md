---
name: reply
description: "Reply to unresolved Foundry annotations on a document. Reads all submitted/replied annotations, groups by section, writes thoughtful responses. Does NOT resolve annotations — that's /update's job. Use when the human asks to respond to feedback, address comments, reply to annotations, or catch up on threads."
argument-hint: "[doc path]"
---

# /reply — Respond to Annotations

Reply to unresolved annotations on a Foundry document. This is a conversation — you read feedback, respond thoughtfully, and surface where agreement exists. You do NOT resolve annotations or edit the document.

## Steps

1. **Determine the target doc.** If `$ARGUMENTS` is provided, use it as the Foundry doc path. Otherwise, check `next.md` for the active design doc and use that. If neither, ask the human which doc to reply on.

2. **Fetch the doc.** Call `mcp__foundry__get_page` to understand the full document structure and content. Read it before responding to any annotations — you need the context.

3. **List unresolved annotations.** Call `mcp__foundry__list_annotations` with `status: "submitted"` and again with `status: "replied"`. Combine the results. If there are no unresolved annotations, say so and exit.

4. **Load full threads.** For each top-level annotation, call `mcp__foundry__get_annotation` to get the full thread (parent + replies). Skip annotations where:
   - The AI already posted the most recent reply and there's no subsequent human response (don't double-reply)
   - The annotation is orphaned or in draft status

5. **Group by section.** Organize annotations by their `heading_path`. Work through the doc systematically, section by section.

6. **Reply to each annotation.** For each thread that needs a response:
   - Re-read the section content the annotation is attached to
   - Read the full thread context (original comment + all replies)
   - Write a thoughtful reply that addresses the specific concern
   - Reference code, design rationale, or trade-offs where relevant
   - If you agree with the feedback, say so clearly — this makes it a candidate for `/update`
   - If you disagree or have concerns, explain why
   - Call `mcp__foundry__create_annotation` with `parent_id` set to the annotation you're replying to

7. **Summarize.** After all replies are posted, output:
   - How many threads were addressed, grouped by section
   - Which threads look like they have agreement (candidates for `/update`)
   - Which threads are still open or need human input
   - Any annotations that were skipped and why

## Rules

- Reply quality matters more than speed. Read the section content before responding.
- One reply per thread, unless the thread has multiple distinct concerns.
- **Do NOT call `resolve_annotation`.** Resolution happens in `/update` after agreement is confirmed.
- **Do NOT call `update_section`.** Doc edits happen in `/update`.
- If you agree with a comment, say so explicitly — the human and `/update` need clear signal.
