# Claymore Plugins

Skills and commands for the **CSDLC methodology** (Claymore Software Development Lifecycle) — a framework for human-AI collaborative software development.

Built for [Claude Code](https://claude.ai/claude-code). Powered by [Foundry](https://github.com/danhannah94/foundry) as the knowledge layer.

---

## Install

```bash
# Add the marketplace
/plugin marketplace add danhannah94/claymore-plugins

# Install the plugins
claude plugin install csdlc@danhannah94-claymore-plugins --scope user
claude plugin install foundry@danhannah94-claymore-plugins --scope user

# Activate
/reload-plugins
```

No clone needed. CC handles caching automatically.

### Scopes

| Scope | Config file | Use case |
|-------|-------------|----------|
| `user` | `~/.claude/settings.json` | Personal, across all projects |
| `project` | `.claude/settings.json` | Team-shared, committed to git |
| `local` | `.claude/settings.local.json` | Project-specific, gitignored |

---

## Prerequisites

- **Claude Code** installed and authenticated
- **Foundry** instance running (local Docker or deployed) — Foundry is the knowledge layer for CSDLC and is required for full functionality
- **GitHub CLI** (`gh`) authenticated — used by `/issue` and sub-agent PR workflows

---

## Commands

### Session

| Command | What it does |
|---------|-------------|
| `/standup [persona]` | Start a CSDLC session. Load context from Foundry, deliver 5-part standup. Optional persona (e.g., `/standup clay`). |
| `/retro` | End session. Write handoff to `NEXT.md` in Foundry. |

### Design

| Command | What it does |
|---------|-------------|
| `/doc <feature>` | Generate a Step 0 design doc. Selects template level (project, epic, subsystem). |
| `/refine <target>` | Challenge and refine any artifact — design doc, idea, issue, plan. |
| `/bootstrap [project]` | Set up CSDLC for a project — generate WORKFLOW.md, create Foundry space. |

### Execution

| Command | What it does |
|---------|-------------|
| `/breakdown <doc>` | Decompose a design doc into implementable stories (Step 1). |
| `/execute <epic>` | Work stories with sub-agents, review each PR, wait for approval before merging. |
| `/intern <story>` | Hand off a single story to a sub-agent. |
| `/shipit <epic>` | Run all stories in an epic autonomously — the Hurricane Pattern. |

### Quality

| Command | What it does |
|---------|-------------|
| `/review <PR>` | Structured review checklist — scope, tests, acceptance criteria. |
| `/qa <story>` | QA against acceptance criteria with evidence. |
| `/present <scope>` | Package completed work for async human review. |

### Ops

| Command | What it does |
|---------|-------------|
| `/issue <title> [--repo owner/repo]` | Create a GitHub issue from context. |

---

## Plugins

### CSDLC (`csdlc/`)

The core methodology plugin. Covers the full CSDLC pipeline:

```
Refinement (continuous)
    ↓
Step 0: Design Doc        → /doc
Step 1: Story Breakdown    → /breakdown
Step 2: Agent Prompts      → (internal, used by /execute and /intern)
Step 3: Execution          → /execute, /intern, /shipit
Step 4: Review             → /review
Step 5: QA                 → /qa
Step 6: Human Review       → /present
```

**Skills:** 13 total — `csdlc-standup`, `csdlc-retro`, `csdlc-bootstrap`, `csdlc-doc`, `csdlc-breakdown`, `csdlc-execute`, `csdlc-intern`, `csdlc-shipit`, `csdlc-review`, `csdlc-qa`, `csdlc-present`, `csdlc-issue`, `csdlc-craft-agent-prompt`

**Bundled templates:** Project design, epic design, subsystem design, workflow, agent-base, doc-agent

### Foundry (`foundry/`)

The knowledge layer plugin. Skills for working with Foundry's MCP toolset.

**Skills:** `foundry-mcp-reference` (tool patterns + gotchas), `foundry-review` (annotation conventions for refinement)

---

## How It Works

CSDLC is a methodology for human-AI collaborative software development. The key ideas:

1. **The process activates on demand.** Run `/standup` to start a CSDLC session. Without it, CC works normally.
2. **Foundry is the knowledge layer.** Design docs, session state (`NEXT.md`), stories, and reviews all live in Foundry. This solves the cold start problem — every session starts informed.
3. **Skills are Foundry-native.** Outputs write to Foundry (annotations, sections, reviews). Inputs read from Foundry.
4. **Refinement is the core practice.** `/refine` challenges any artifact before it's acted on. 30 minutes of refinement saves hours of rework.
5. **Sub-agents (interns) do the implementation.** The AI Lead orchestrates; sub-agents execute in isolated worktrees.

For the full methodology, see the [CSDLC Thesis](https://foundry-claymore.fly.dev).

---

## Project Structure

```
claymore-plugins/
├── .claude-plugin/
│   └── marketplace.json        # Plugin registry
├── csdlc/
│   ├── .claude-plugin/
│   │   └── plugin.json         # CSDLC plugin manifest
│   ├── commands/               # Slash command shortcuts
│   │   ├── standup.md
│   │   ├── retro.md
│   │   ├── doc.md
│   │   ├── refine.md
│   │   ├── bootstrap.md
│   │   ├── breakdown.md
│   │   ├── execute.md
│   │   ├── intern.md
│   │   ├── shipit.md
│   │   ├── review.md
│   │   ├── qa.md
│   │   ├── present.md
│   │   └── issue.md
│   └── skills/                 # Skill implementations
│       ├── csdlc-standup/
│       ├── csdlc-retro/
│       ├── csdlc-bootstrap/
│       │   └── templates/      # Bundled workflow template
│       ├── csdlc-doc/
│       │   └── templates/      # Bundled design doc templates
│       ├── csdlc-breakdown/
│       ├── csdlc-execute/
│       ├── csdlc-intern/
│       ├── csdlc-shipit/
│       ├── csdlc-review/
│       ├── csdlc-qa/
│       ├── csdlc-present/
│       ├── csdlc-issue/
│       ├── csdlc-craft-agent-prompt/
│       │   └── templates/      # Bundled agent templates
│       └── csdlc-process/      # Methodology reference
│           ├── PROCESS.md
│           └── CORE_VALUES.md
└── foundry/
    ├── .claude-plugin/
    │   └── plugin.json         # Foundry plugin manifest
    └── skills/
        ├── foundry-mcp-reference/
        └── foundry-review/
```

---

## Contributing

This is an active project. The methodology evolves with every session.

- **Issues:** Use `/issue` (meta, right?) or file directly on GitHub
- **Design doc:** Full architecture at `projects/claymore-plugins/design` on [Foundry](https://foundry-claymore.fly.dev)

---

*The Claymore way: ship fast, ship right, get better every time.*
