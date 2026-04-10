---
name: csdlc-bootstrap
description: Set up CSDLC for an existing project — generate WORKFLOW.md, create Foundry project space, scaffold a design doc. Use when the user says "bootstrap", "init", "set up CSDLC", "onboard this project", or when /standup detects no NEXT.md and no WORKFLOW.md.
process_version: "1.0"
---

# CSDLC Bootstrap

Set up the CSDLC methodology for a project. This is the "first 5 minutes" experience — take an existing project (or new one) and give it the structure CSDLC needs to operate.

## Procedure

1. **Gather project info.** Ask the user (skip questions they've already answered):
   - What's the project name? (e.g., "Foundry", "Routr", "QuoteAI")
   - What's the 3-letter project prefix? (e.g., FND, RTR, QAI — used for `{PROJECT}-{EPIC#}-{STORY#}` naming)
   - What repo(s) does it live in? (absolute path on disk)
   - What's the tech stack? (language, framework, database, deployment)
   - Is there an existing design doc, README, or architecture description?
   - What's the Foundry instance URL? (e.g., `https://foundry-claymore.fly.dev`)

2. **Generate WORKFLOW.md.** Use the bundled `templates/workflow.md` template. Fill in the project-specific details from step 1. Write it to the project repo root.

3. **Create Foundry project space.** Using Foundry MCP tools:
   - `create_doc('projects/{project-name}/design', template='blank', title='{Project Name} — Design Doc')`
   - If the user has existing architecture docs, offer to populate the design doc scaffold from them.

4. **Set up NEXT.md.** If no `status/next.md` (or `status/{username}/next.md`) exists in Foundry:
   - Create it with the canonical NEXT.md format from the `csdlc-retro` skill.
   - Populate the Priority Stack with the current project.
   - Leave SHIPPED empty — this is day one.

5. **Update CLAUDE.md (optional, only if user asks).** Suggest adding: "This project uses CSDLC — start work sessions with `/standup`."

6. **Report what was created.** List every file and Foundry page created, with paths.

## What NOT to do

- Don't overwrite existing WORKFLOW.md, CLAUDE.md, or design docs without asking.
- Don't create empty placeholder docs "just in case" — only create what's needed now.
- Don't start working on the project after bootstrap — this skill sets up the structure, then stops. The user decides what to work on via `/standup`.
