---
description: Refine anything — a design doc, an idea, an issue, a plan. Challenge assumptions, flag gaps, resolve ambiguity.
argument-hint: <doc path, issue #, or describe the idea>
---

Refine this: **$ARGUMENTS**

You are in **CSDLC Refinement mode**. Your job is to challenge the thing
above before anyone acts on it. Resolve ambiguity, don't paper over it.
Refinement isn't a step — it's the core practice. It applies to everything.

## Step 1: Figure out what you're refining

`$ARGUMENTS` could be anything:

- **A file path** (local or Foundry) → read it
- **A GitHub issue** (`#123` or `owner/repo#123`) → fetch it with `gh`
- **A URL** → fetch and read it
- **An inline idea** (just text, no path) → the argument IS the thing to refine
- **Nothing** → ask the user what they want to refine

Don't guess. If the input is ambiguous, ask one clarifying question.

## Step 2: Understand the intent

Before running a checklist, understand what this thing IS:

| If it's a... | Focus refinement on... |
|---|---|
| Design doc | Architecture, scope, stories, open questions |
| Feature idea | Problem clarity, feasibility, scope, MVP definition |
| Bug / issue | Root cause clarity, repro steps, fix scope, blast radius |
| Plan / roadmap | Priorities, dependencies, sequencing, what's missing |
| PR or diff | Intent clarity, edge cases, test coverage, naming |
| Process / workflow | When it applies, what it replaces, failure modes |

Adapt the checklist to the thing. Don't ask "what's the rollback plan?" for
a half-baked idea that's still in the "should we even do this?" phase.

## Step 3: Run the refinement checklist (adapted to context)

For each category, state what's clear and what's not:

- **Goal clarity** — Can you state success in one sentence? Is the problem
  concrete, or does it dissolve into buzzwords?
- **Scope** — What's explicitly OUT of scope? If nothing is, that's a red
  flag. What's the smallest version that still delivers value?
- **Assumptions** — List every assumption being made. Which are load-bearing?
  Which are guesses dressed as facts?
- **Dependencies** — What has to be true elsewhere for this to work?
- **Risks** — What breaks if this is wrong? Is it reversible? What's the
  blast radius?
- **Measurement** — How will we know it worked? What's the signal?
- **Unknowns** — What does this pretend to know that it doesn't?

Skip categories that genuinely don't apply — but think twice before skipping.
The ones that feel irrelevant are often the ones hiding the real risk.

## Step 4: Push back honestly

If something is vague, say so. If an assumption seems wrong, flag it. Don't
soften the feedback — authenticity over comfort. Cite specifics when possible.

## Step 5: Output a refinement report

```
## Refinement Report: <title or summary>

### ✅ What's solid
- <list>

### 🟡 What needs clarification
- <specific question> (<context>)
- ...

### 🔴 What looks wrong or risky
- <the concern> (<why it matters>)
- ...

### Recommended next steps
1. <action>
2. ...
```

## Hard rules

- **Do NOT implement anything.** This is a thinking step, not an action step.
- **Do NOT modify any files** unless the user explicitly says to.
- **Understand before critiquing.** Read the full thing first. Skim-refining
  produces garbage feedback.
- **Don't skip the checklist** because it feels bureaucratic. The checklist
  exists because without it, important questions get missed.
- **Match depth to maturity.** A napkin-sketch idea gets broad-stroke feedback.
  A 500-line design doc gets line-by-line scrutiny.
