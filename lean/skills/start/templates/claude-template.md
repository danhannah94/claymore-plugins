# CLAUDE.md — `<project-name>`

*Project-specific AI instructions. Read on every `/start <project>`.*

## What this project is

*One paragraph. What the project does and the core value proposition. Link to the design doc.*

- Design doc: [Foundry path or local path]

## Session commands

- `/start <project>` — bootstrap session scoped to this project (reads this project's `next.md`, `decisions.md`, and this file)
- `/stop <project>` — end session, write handoff to this project's `next.md`
- `/reply <foundry-doc-path>` — reply to annotations on a Foundry design doc
- `/update <foundry-doc-path>` — lock in agreed decisions from annotation threads

## Repo conventions

*Anything non-obvious about how code is organized, named, or tested in this project.*

- Package manager: `<pnpm | npm | yarn>`
- Monorepo tooling: `<pnpm workspaces | turborepo | none>`
- Language/framework: `<Next.js / TypeScript | etc.>`
- DB client: `<Drizzle | Prisma | pg>`
- Test runner: `<vitest | jest>`

## Common commands

*Day-to-day commands the human and AI run in this project.*

- `pnpm dev` — start dev server
- `pnpm test` — run tests
- `pnpm <custom>` — <what it does>
- `make db-reset` — wipe and rebuild local DB

## MCP servers used

*Which MCP servers this project expects to be configured.*

- `foundry` — design docs, annotations, refinement
- `<project-specific MCP>` — what it provides

## Code conventions

*Any coding standards or patterns specific to this project.*

- Validate all external inputs with Zod before touching the DB
- Use `<convention>` for <topic>
- <etc.>

## Pointers

- Design doc: `<path>`
- Epic docs: `<path>/epics/`
- Project `next.md`: this project's handoff file (updated each `/stop <project>`)
- Project `decisions.md`: active decisions (updated during `/stop <project>` triage)

---

*This file lives at `<workspace>/<project>/CLAUDE.md`. Update it as conventions emerge or change.*
