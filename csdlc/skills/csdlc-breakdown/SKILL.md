---
name: csdlc-breakdown
description: Decompose a design doc into implementable stories (CSDLC Step 1). Read a design doc from Foundry or local file, break it into scoped stories with acceptance criteria. Use when the user says "breakdown", "stories", "decompose", "Step 1", or "break this into tickets".
process_version: "1.0"
argument-hint: <design doc path (Foundry or local)>
---

# CSDLC Breakdown — Step 1 Story Decomposition

Break down this design doc into implementable stories: **$ARGUMENTS**

Step 1 takes a refined design doc and extracts implementable stories. By the time this runs, most ambiguity should already be resolved from Step 0 refinement.

## Procedure

1. **Read the design doc.** `$ARGUMENTS` is either:
   - A Foundry path → `get_page` to read it
   - A local file path → read the file
   - Nothing → ask the user which design doc to break down

2. **Identify the project prefix.** Check the design doc or WORKFLOW.md for the naming convention (e.g., `FND` for Foundry, `RTR` for Routr). If not found, ask.

3. **Extract stories.** For each distinct capability, feature, or change in the design doc, create a story. Every story needs:

   - **ID:** `{PROJECT}-{EPIC#}-{STORY#}` (e.g., `FND-E12-S1`)
   - **Title:** One-line summary of what this story delivers
   - **Scope:** What specifically gets built or changed. Be concrete — name files, components, endpoints.
   - **Acceptance criteria:** Specific, testable, unambiguous. Each criterion is a checkbox.
   - **Target files/areas:** Where the work happens. Prevents wrong-target bugs.
   - **Boundaries:** What NOT to touch. Explicit "do not modify" rules.
   - **Dependencies:** Which other stories must complete first? Which can run in parallel?
   - **Complexity:** S / M / L (gut-check, not sprint planning)

4. **Sequence the stories.** Identify:
   - **Serial dependencies:** S2 requires S1 to be merged first
   - **Parallel groups:** S3 and S4 can run simultaneously
   - **Foundation stories:** Must go first because everything depends on them

5. **Output the breakdown.** Format as a structured list. Include a dependency diagram if there are non-trivial dependencies.

## Story format

```markdown
### {PROJECT}-{EPIC#}-S{N}: {Title}

**Scope:** {What specifically gets built}

**Acceptance criteria:**
- [ ] {Specific, testable criterion}
- [ ] {Another criterion}

**Target files:** {file1.ts}, {file2.ts}

**Boundaries:** Do NOT modify {file3.ts} or {subsystem}.

**Dependencies:** Requires S{N-1}. Can parallel with S{M}.

**Complexity:** S / M / L
```

## Where to save

Present the breakdown in conversation first. Then offer:
- **Foundry** (preferred) — write stories as sections on the epic design doc using `update_section` or `insert_section`
- **Local file** — write as a `.md` file
- **Both**

## Principles (from PROCESS.md)

- **One story = one deliverable.** If a story has two unrelated concerns, split it.
- **Stories are independently implementable** (or with explicit, stated dependencies).
- **Acceptance criteria are testable.** "Works correctly" is not a criterion. "Returns 401 on expired token" is.
- **Boundaries prevent scope creep.** Explicitly stating what NOT to touch is as important as stating what to do.
- **Refinement applies here too** — but it's faster because Step 0 already resolved most ambiguity.
