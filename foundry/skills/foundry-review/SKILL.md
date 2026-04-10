---
name: foundry-review
description: Foundry-specific conventions for reviewing and annotating documents. How to refine docs via Foundry's annotation system — highlighting, threading, resolving. Use when refining a Foundry doc, reviewing annotations, or when the /refine command targets a Foundry page.
process_version: "1.0"
---

# Foundry Review — Annotation Conventions

When refining or reviewing a document in Foundry, follow these conventions for annotations. This skill is invoked automatically by `/refine` when the target is a Foundry doc.

## Annotation principles

- **Always highlight text.** When creating an annotation, use `quoted_text` to reference the specific text you're commenting on. Annotations without highlighted text are harder to trace back to the content.
- **Prefer many short comments over few long ones.** Each comment should address ONE concern. Don't bundle 5 questions into a single annotation — they'll get tangled in the reply thread.
- **Use threads for back-and-forth.** Reply to annotations using `parent_id` rather than creating new top-level annotations. Keep the conversation attached to the original concern.
- **Resolve when agreed.** Once both parties agree on the resolution, use `resolve_annotation`. Don't leave resolved items as submitted — it clutters the view.
- **Don't edit the doc to address feedback — annotate first.** The annotation thread IS the refinement record. Edit the doc only after the feedback is discussed and agreed upon.

## Review workflow

1. **Read the full page.** `get_page` to understand the complete doc before commenting. Don't skim-review.
2. **Create a review.** Use `submit_review` to group your annotations into a coherent review batch.
3. **Annotate section by section.** For each concern:
   - Highlight the relevant text (`quoted_text`)
   - Write a focused comment (one concern per annotation)
   - Use the appropriate `section` field to anchor the annotation
4. **Check for existing annotations.** `list_annotations` before starting — don't duplicate feedback that's already been given.
5. **Process replies.** When the other party responds:
   - If you agree → `resolve_annotation`
   - If you need to discuss further → reply to the thread (`parent_id`)
   - If you disagree → explain why in a reply, don't just re-assert

## Annotation types by context

| Context | Focus | Tone |
|---------|-------|------|
| Design doc refinement | Architecture, scope, assumptions, risks | Challenging — poke holes |
| Story review | Acceptance criteria, completeness, clarity | Precise — is this implementable? |
| QA report | Evidence, coverage, edge cases | Factual — show the receipts |
| Code review (via Foundry) | Intent, approach, alternatives | Constructive — suggest, don't just criticize |

## Anti-patterns

- **Wall-of-text annotations.** If your comment is more than 4-5 sentences, split it into multiple annotations on different highlighted sections.
- **Annotations without quotes.** Always highlight. "I have a concern about this section" — which section? Point at it.
- **Orphaned threads.** Don't abandon a thread without resolving or explicitly deferring. Unresolved annotations are open questions that block progress.
- **Editing the doc to "fix" feedback.** The annotation is the discussion. The doc edit is the outcome. Don't skip the discussion.
