# hl — Hannah Labs methodology (v3)

Third iteration of a collaborative-development methodology. Previously OpenClaw, then [lean](../lean/). Same goal: **low ceremony, high signal.**

## What's in this plugin

- **`/hl:start`** — bootstrap a session (reads `next.md` + `methodology.md` + `decisions.md`, surfaces deferred red-team gaps, gives orientation)
- **`/hl:stop`** — end a session cleanly (triages decisions, surfaces unresolved red-team gaps, writes `next.md` for the next session)
- **`/hl:red-team`** — stress-test design docs (load → identify gaps by severity → triage interactively → apply locked decisions → recap)
- **`/hl:blue-team`** — make scope cuts from a red-team menu and produce the requirements doc (aggregate → sort → triage → write contract → reconcile)
- **`/hl:methodology`** — read the v3 methodology reference

## Pipeline

```
draft → red-team → blue-team → (epic refinement) → reply → update → ship
```

- **Draft**: rough doc(s) in Foundry (epic drafts, plans). Manual for now.
- **Red-team** (`/hl:red-team`): adversarial review surfaces gaps and decisions. Tightens drafts.
- **Blue-team** (`/hl:blue-team`): scope cuts from the menu. Produces the requirements contract.
- **Epic refinement**: reconcile drafts with the requirements contract.
- **Reply / Update**: currently delegated to `/lean:reply` and `/lean:update` until hl variants ship.
- **Ship**: code that implements the design.

## Why this pipeline shape

Subtraction is easier than addition. Red-team surfaces a comprehensive landscape; blue-team makes scope cuts from a position of total visibility. You can't write good requirements for something you've never imagined — but you CAN imagine the landscape adversarially, then prune. Divergent-then-convergent.

## Installation

This plugin is published via the `claymore-plugins` marketplace:

```
/plugin marketplace add danhannah94/claymore-plugins
/plugin install hl@claymore-plugins
```

## Why a new plugin instead of extending lean?

The lean methodology (currently at v0.2.0) is a frozen reference point — it represents a working state of the process that other artifacts depend on. v3 is a fresh evolution that introduces:

- **An explicit pipeline shape** with stages and hand-offs
- **Red-team + blue-team as paired skills** for divergent-then-convergent scope work
- **Requirements docs as the scope contract** per initiative
- **Red-team gap continuity** between sessions — unresolved items get surfaced on the next `/hl:start`
- **Amendments first-class** — scope changes are logged, not silent

Existing lean users can keep using lean. v3 is opt-in.

## Versioning

- `0.1.0` — initial release with start, stop, red-team, methodology
- `0.2.0` — added blue-team skill, updated methodology to reflect the locked pipeline shape with red/blue pair

## License

MIT. © Dan Hannah / Hannah Labs.
