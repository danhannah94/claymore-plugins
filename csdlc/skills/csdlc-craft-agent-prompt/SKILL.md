---
name: csdlc-craft-agent-prompt
description: Craft a sub-agent prompt from a refined CSDLC ticket, mirroring PROCESS.md Step 2 (Agent Prompt Crafting). Use when the user says "craft the agent prompt", "dispatch this ticket", "send this to CC", "spin up a sub-agent", "hand this off", or when a ticket is refined to ready-for-execution state. This is the translation layer between refinement (AI Lead) and execution (sub-agent) — trigger proactively once a ticket is clearly ready.
---

# CSDLC Craft Agent Prompt

You're the AI Lead in CSDLC Step 2: Agent Prompt Crafting. A ticket has been refined and is ready to hand off to a Claude Code sub-agent. Translate the ticket into a precise, actionable prompt the sub-agent can execute without back-and-forth.

The sub-agent has **no memory** of refinement, but it **does** have read access to the Foundry MCP connector — it can use `get_page`, `search_docs`, `list_pages`, and `list_annotations` to gather additional context on its own (including private docs). It does **not** have write access: it cannot create or edit Foundry docs, only annotations. Your prompt's job is to point it at the right starting docs and curate the most critical context; the sub-agent can pull the rest as needed.

## Before writing

Gather these. Ask the user if any are missing — a prompt missing context is worse than no prompt.

1. **Ticket ID and doc path** (e.g., `FND-E12-S3` at `projects/foundry/epics/e12-mcp-auth.md#s3`).
2. **Repo + base branch** — the sub-agent needs to know where to work.
3. **Relevant design doc sections** — pull them from the design doc; don't hand over the whole doc.
4. **Project WORKFLOW.md or AGENT_BASE.md** — most CSDLC repos have one. If it exists, your prompt should **reference** it rather than inline its content.
5. **Gotchas relevant to this ticket** — from NEXT.md and the design doc. Inline only the ones that apply.

## Prompt structure

Mirror PROCESS.md > Pipeline > Step 2's six elements. For each element, decide whether to inline the content or delegate to AGENT_BASE.md.

```
# Role and context

You are a sub-agent executing CSDLC ticket <TICKET_ID>.
Repo: <REPO> · Base branch: <BASE_BRANCH> · Worktree: <WORKTREE_PATH (if applicable)>
The ticket doc is at <PATH> — read it first.

# 1. Setup
<If the repo has AGENT_BASE.md: "Read tasks/AGENT_BASE.md for setup, git identity, and branch conventions.">
<Otherwise inline the setup commands.>

# 2. Task context

<One-paragraph ticket summary distilled from the ticket doc.>

<Relevant design-doc excerpts — verbatim, cited by section. Keep to what's needed.>

<Current system state that matters — cite the canonical doc (design doc section, NEXT.md entry, etc.) rather than specific PR numbers. e.g., "the /mcp/sse endpoint requires auth — see methodology/process for current auth model">

# 3. Acceptance criteria

- <Concrete, testable criterion copied from the ticket>
- <...>

# 4. Boundaries (explicit do-not-touch)

- <File / directory / subsystem the agent must not modify>
- <Scope constraints not covered by acceptance criteria>

# 5. Verification

<If AGENT_BASE.md covers build/test commands: "Run the Before Submitting checklist in AGENT_BASE.md.">
<Otherwise inline the specific commands: `npm run build`, `npm test`, any ticket-specific reproduction.>

# 6. Boilerplate

<If AGENT_BASE.md covers commit/PR conventions: "Follow AGENT_BASE.md for commit types, PR body template, and rules.">
<Otherwise inline commit conventions, PR title format, and branch naming.>

# Gotchas (read these, do not rediscover)

- <From NEXT.md + ticket doc, only the ones that apply to this ticket>

# Files to read first (in order)

1. <path> — <why>
2. <path> — <why>
3. <path> — <why>

(Max 5. Unordered = wasted context.)

# Escalation

If you hit something unresolvable in ~30 min of focused work — ambiguous requirement,
pre-existing broken test, missing dependency, unanswered design question — stop, post
a Foundry annotation on the ticket doc describing the blocker, and exit. Do NOT guess
through ambiguity. The AI Lead will re-refine and re-dispatch.
```

## AGENT_BASE.md delegation

If the repo has `tasks/AGENT_BASE.md` (the openclaw/CSDLC convention), do **not** duplicate its content in your prompt. It already covers:

- Project-specific setup (`cd`, git config, branch creation)
- Before-submitting commands (`npm run build`, `npm test`, etc.)
- Commit types table and PR body template
- Style guide and scope rules (1 ticket = 1 PR, do-not-modify scopes)

Your prompt should reference it by path and add **only** the ticket-specific deltas. Sections 1 / 5 / 6 above often reduce to a single line each.

If the repo doesn't have an AGENT_BASE.md yet, flag it to the user and ask whether to create one this session.

## Writing guidance

- **Rewrite for fresh context.** The sub-agent doesn't know what was discussed in refinement. Any context from the conversation must be in the prompt or a doc it points at.
- **Read order matters, 3–5 files max.** Unordered reading lists cause either hallucination (reads nothing) or context bloat (reads everything).
- **Keep acceptance criteria testable.** "Make it faster" fails. "p99 < 200ms on benchmark X" passes. Push back on vague criteria during refinement, not at dispatch time.
- **Don't over-specify the how.** Specify the *what* (behavior, interfaces, criteria) and the *constraints*. Let the sub-agent pick implementation details — it's a capable engineer.
- **Escalation is a feature.** Fast escalation on ambiguity beats flailing, and for bridge v1 (single-mutex) a stuck run blocks the next.

## Output

Return the prompt as a fenced markdown code block the user can copy, OR write it to a file under the workspace and return a link — ask if unclear. Also summarize in chat (not in the prompt itself):

1. Which ticket the prompt is for.
2. Expected artifacts (PR + Foundry annotation on the ticket doc, unless non-standard).
3. **Refinement gaps you filled by inference** — anything vague in the ticket doc that got resolved during prompt-writing. The user should sanity-check these before dispatching.

## Edge cases

- **Missing or vague acceptance criteria:** Don't dispatch. Go back to Step 1 refinement first.
- **References to nonexistent files:** Surface to user — might be intentional (new file) or stale.
- **Ticket bigger than one run:** Split during refinement. If the user insists on shipping as one, include an explicit "stop after N criteria and escalate" clause.
- **Read-only / research ticket:** Same structure, replace "open a PR" with "post findings as a Foundry annotation on the ticket doc."
- **No AGENT_BASE.md and no WORKFLOW.md:** Inline all six elements fully. Flag as tech debt at session end.
