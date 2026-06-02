---
name: ship
description: "Ship a batch of stories through the agentic pipeline. Parses dependencies → computes execution waves via topo sort → spawns parallel feature agents in git worktrees per wave → conditional QA keyed to a per-project QA backend (test-env+Crucible, OR headless-preview+dev-auth) → wave gate → rebase remaining → optional deploy+migrate phase → batch review. Use when shipping a whole epic or any batch of stories; single-story batches work too. The final 'ship' stage of the hl methodology pipeline."
argument-hint: "[path to design doc with Story Summary, or inline batch spec]"
---

# /hl:ship — Batch/Wave Agentic Pipeline (v3)

The **Ship** stage of the hl methodology pipeline (draft → red-team → blue-team → reply → update → **ship**). Orchestrates Agent Prompt Crafting → Sub-agent Execution → AI Lead Review → QA → Review → Ship (→ optional Deploy) over a **batch** of stories. Uses **git worktrees** for parallel execution, **topological sort** for wave scheduling, and **conditional QA** keyed to each story's `visual_surface` flag and the project's QA backend.

The orchestrator (you) is the glue. Drafting + story breakdown (with explicit acceptance criteria) must already be done before `/hl:ship` is invoked — this is downstream of red-team/blue-team. If a story is ambiguous mid-run, stop and refine; don't have agents guess.

## Kickoff config (agree with the human before Phase 0)

Three knobs are set per batch — confirm them up front (sensible defaults in **bold**):

- **QA backend** — how visual stories get verified. See [Phase 1d](#1d--conditional-qa). Either **(a) test-env + Crucible** (a per-branch Docker harness exists) or **(b) headless-preview + dev-auth** (no test-env; app behind auth — drive Claude Preview against a worktree dev server). Detect from the project; if neither fits, ask.
- **Merge authority** — **human-merge** (orchestrator presents each wave, human clicks merge) OR **delegated** (human grants the orchestrator merge authority for the batch: merge when QA is clean, post per-wave batch summaries for transparency, don't block on a click). Delegation is great for momentum on a trusted batch; default to human-merge unless the human says otherwise.
- **Agent model** — **inherit/Sonnet** to economize, or **Opus** for max quality on every feature/QA agent. The human picks based on stakes.

## Inputs

Two forms accepted as `$ARGUMENTS`:

### Form A — design-doc path with Story Summary table
Path to an epic doc containing a `## Story Summary` section with columns: `Story | Title | Scope | Deps | Visual | Phase`. `/hl:ship` parses the table, cross-references each row against the doc's `## Stories` sections for acceptance criteria, and builds the batch.

### Form B — inline batch spec (YAML-ish)
```
repo: /abs/path/to/repo
stories:
  - id: S1
    title: Schema + DAOs
    branch: feat/e12-s1-schema
    deps: []
    visual_surface: false
    spec: projects/<project>/epics/<epic>.md#<story-id>
  - id: S2
    ...
```

Either form produces the internal batch object. If neither is parseable, stop and surface the expected format. Do not guess.

**Dependency-encoding tip:** deps drive the wave topo-sort. Use them not just for *code* dependencies but to **serialize stories that touch the same files** (e.g. two stories both editing the app shell/header). Worktrees isolate implementation, but same-file edits collide at rebase — a dep puts them in different waves and dodges the conflict. (EPIC-5: making the notifications story depend on the settings story avoided a guaranteed header conflict.)

## Phase 0 — Preflight

1. **Sync main.** `cd <repo> && git checkout main && git pull --ff-only`. Abort if uncommitted changes or divergent main.
2. **Main-green gate — by the project's ACTUAL CI contract.** Run what the project treats as CI (e.g. `scripts/ci.sh`, not a blanket `npm test`). **If that is red, stop and offer to fix first** — wave stories inherit failures and nothing downstream is trustworthy. **But** distinguish the CI contract from *out-of-CI* suites: a project may deliberately exclude slow/infra-heavy integration tests from CI (autri's `ci.sh` excludes the Postgres-backed app suite). If an out-of-CI suite is red for a reason **unrelated** to this batch (e.g. a dependency bump broke it), that is NOT a batch blocker — flag it, spawn a fix-it task, and proceed; just don't let feature agents chase it (tell them in their prompts). Establish the per-story green baseline the agents must hold (typically `typecheck` + the project's CI), since the heavy suite may not be runnable.
3. **Deploy-env readiness gate.** For any story adding `throw`-on-boot env-var requirements (fail-loud startup checks, required secrets), verify the deploy target has them set BEFORE merging. Cross-reference `next.md` / project notes for known-unset secrets. Skipping this has taken a prod app down before.
4. **Routing-layer-above-app check.** When a story introduces new HTTP paths, verify the whole stack above the app routes them: proxy scripts, reverse proxies, CDN allow-lists (CloudFront origin-request header caps!), `fly.toml`, ingress. "Route mounted in the app" is necessary, not sufficient.
5. **Clean stale worktrees.** `git worktree prune`, then scan `<repo>.worktrees/` for dirs not in `git worktree list` and delete them.
6. **Validate batch.** Every `deps` references an id in the batch; no cycles; every story has id, title, unique branch, deps, visual_surface, spec.
7. **Compute waves.** Kahn's algorithm: stories with no unmet deps form wave N; remove; repeat. Stop on cycle.
8. **Present wave plan to human.** Table of waves (per-story id, title, branch, visual flag) + totals (stories, waves, PRs, visual:backend ratio). **Wait for approval before firing anything** — a wrong wave plan wastes a lot of agent time. (Under delegated merge authority this is still the one up-front gate.)

## Phase 1 — Execute each wave

Loop over waves. Within each wave:

### 1a — Spawn feature agents in parallel worktrees
For each story: create worktree `git worktree add <repo>.worktrees/<branch-slug> -b <branch> main` (slug = branch with `/`→`-`). Spawn a sub-agent (model per kickoff) with a self-contained prompt. The agent:
- Works inside the worktree path (explicit `cd` — **never** the main checkout); runs `pnpm install` there first if the project uses workspaces (worktrees start without `node_modules`).
- Implements against the spec + AC.
- Runs local verification (build, typecheck, and its OWN targeted tests per project convention). **Tell it explicitly** about any known-red out-of-CI suite so it doesn't chase it; have it run new tests in isolation (mocking the auth boundary if needed).
- Commits with a WHY message; pushes; opens a PR (story id, what changed, files, AC-to-diff mapping, QA hints — including *how to reach/trigger the change* for the QA agent).
- Returns PR URL + 2-3 line summary + caveats.

**Parallelism:** no hard cap — orchestrator judgment by wave size + machine headroom. Fire independent stories together.
**Do not** have feature agents boot test-envs or QA harnesses. QA agents do that.

### 1b — Wait for the wave's feature agents
Don't review until every story returns a PR URL or a clean failure (partial-wave review → rebase chaos). On agent failure: leave the worktree, record it, surface with retry/re-scope/skip options.

### 1c — Orchestrator code review (per PR)
`gh pr diff <N>` and check: security, correctness, obvious bugs, spec adherence, commit-message quality, scope-boundary violations. **Security-sensitive stories (auth, prod-DDL, anything that could open a prod hole) get the closest read** — e.g. verify a dev-only auth bypass is statically impossible in prod, not just runtime-gated.
- **Nits** → spawn a **cleanup sub-agent** at the same worktree with line-referenced fixes (don't line-edit yourself — orchestrator does judgment, not edits). It pushes a commit; re-glance; proceed.
- **Structural issues** (bugs, security, missed AC) → pause that PR, surface; let the rest of the wave continue through QA.
- **Clean** → 1d.

### 1d — Conditional QA
Partition the wave's PRs by `visual_surface`. **Backend PRs** gate on the project's CI contract (`gh pr checks`, or run `ci.sh`) + code review — the test suite *is* their QA; no visual QA. **Visual PRs** use the kickoff **QA backend**:

**Backend (a) — test-env + Crucible** (a per-branch Docker harness exists, e.g. Foundry):
One feature-QA sub-agent for the whole wave. It iterates PRs serially: boot test-env (`test-env/scripts/qa.sh <branch>`), verify against AC, collect verdict + baseline recommendations, tear down before the next. On per-PR PASS it posts evidence to the PR; on ISSUES_FOUND/NEEDS_HUMAN it records for the report. Orchestrator reviews the consolidated baseline recommendations and approves them in one batch (`mcp__crucible__approve_baseline`) — **never** auto-approve from the agent. (One agent per wave because Docker boots are serial anyway; N parallel QA agents don't save wall-clock and fragment context.)

**Backend (b) — headless-preview + dev-auth** (no test-env; app behind auth, e.g. autri):
Pre-merge visual QA is a sub-agent driving the **Claude Preview MCP tools** (`preview_start`, `preview_snapshot`/`screenshot`/`resize`/`fill`/`click`, `preview_console_logs`) against the branch's **worktree dev server**. Sub-agents *can* drive Preview — run them **serially** (the preview server tracks one dev server; one QA agent at a time, which also matches a shared local DB/port). Key requirements:
- **A dev-only auth bypass is a prerequisite** for reaching authed pages headlessly. If the app gates routes behind a real IdP (Cognito/OAuth/etc.), real login isn't headless-driveable. The fix: an env-gated dev credentials path, **double-gated and statically impossible in prod** (e.g. `NODE_ENV !== 'production' && DEV_AUTH_FLAG`). If it doesn't exist yet, make it **story-zero** in wave 0 — every visual story's QA depends on it. Validate it for real once (it's the linchpin) and capture the exact headless-login recipe to paste into every later QA-agent prompt.
- The QA agent creates the worktree's `.env.local` (DB URL, dev-auth flag, a throwaway AUTH_SECRET), boots `preview_start`, logs in via the recipe, verifies each AC (use `preview_resize` for responsive/viewport stories; confirm dark mode if that's the default; check `preview_console_logs`), screenshots evidence, tears down.
- Returns a per-PR verdict table + evidence. Crucible is NOT the pre-merge gate here — it's the **post-deploy** prod regression (Phase 2.5).

Either way: on **ISSUES_FOUND** with a localized cause, fix-forward via a **cleanup sub-agent** at the worktree (re-verify after), then proceed — don't merge a known-failing AC.

### 1e — Wave review + merge
Present the wave: per-PR URL, story id, verdict, anything to eyeball, recommended merge order.
- **Human-merge:** wait for approval; handle per-PR (human may approve some, hold others). Do not merge autonomously.
- **Delegated:** merge each PR whose QA + review are clean; post the wave summary for transparency but don't block on a click. Hold + surface any PR with an open structural issue.

### 1f — Merge + rebase remaining
Merge one at a time. After each: pull main; for each unmerged wave PR `cd <worktree> && git rebase origin/main` → if clean `git push --force-with-lease`, if conflict **surface** (don't auto-resolve); re-run checks before the next merge. Two wave-mates conflicting means a missed dep or a scope overlap — both warrant a pause. **Never `--no-verify` or force-push to main.** (The benign exception: `gh pr merge --delete-branch` fails to delete the *local* branch when it's checked out in a worktree — harmless; the remote branch is deleted and the worktree is cleaned in 1i. A `--no-verify` on a *shell-only* change in a worktree with no `node_modules`, where the hook only gates TS, is a judgment call — note it.)

### 1g — Wave-level regression (test-env backend only)
If the QA backend is (a) test-env AND any wave story was visual: one sub-agent boots a fresh test-env from main, runs the full baseline sweep, tears down, returns PASS/ISSUES_FOUND/NEEDS_HUMAN. ISSUES_FOUND → pause the batch. **For backend (b)**, there is no pre-merge regression env — regression happens post-deploy against prod (Phase 2.5); the headless-preview pre-merge QA already validated each PR against the merged code.

### 1h — Post-merge smoke (PRs that add HTTP surface)
Curl new endpoints locally AND (after deploy) at the prod URL — a proxy/CDN/ingress layer can swallow routes that pass local smoke.

### 1i — Worktree cleanup
After each merged PR: `git worktree remove <repo>.worktrees/<branch-slug>` (`--force` if dirty), then `git worktree prune`. `git branch -D` the merged local branch.

### 1j — Wave gate
Before wave N+1: all wave-N PRs merged or explicitly skipped; regression green (backend a) or pre-merge QA clean (backend b); main clean + CI green; worktrees cleaned. If any gate fails, pause and surface.

## Phase 2 — Final report
Wave-by-wave table (stories, PRs, merge order, QA verdicts); baselines updated; regression results; stories skipped/deferred (with reason); open follow-ups surfaced (spawn tasks for out-of-scope finds); then either proceed to deploy (Phase 2.5) or suggest `/hl:stop`.

## Phase 2.5 — Deploy + migrate (optional — only if the human asked to ship to prod this run)

Deploying is outward-facing and hard to reverse. Even with standing authorization, gate each prod-mutating command on a clean diff and halt if anything looks wrong.

1. **Discover the prod migration path BEFORE deploying — do not improvise prod DDL.** If a story added a DB migration, find how migrations actually reach prod (it is often NOT in the app repo / deploy script). **The runner may already exist, hidden in infra** — in autri it was a CDK custom-resource Lambda invisible from the app repo; CloudWatch logs were the audit trail. Investigate (read infra, git history, decisions docs; read-only AWS describes) before building anything. Private-subnet DBs have no laptop path — the established in-VPC compute is the migration host.
2. **Migrate first (schema-first ordering).** Migrations are additive and must land before the app/worker code that references the new columns. Run the project's migration mechanism (and `diff` the migration deploy first — confirm it touches *only* the migration trigger/asset, nothing on DB/SG/buckets). Verify it applied (e.g. the runner's logs: `applied:N`).
3. **Deploy the app.** Use the project's deploy script. Long Docker builds: run **backgrounded + sandbox-disabled** (build can exceed the foreground cap) and **prune images+cache first** (NOT volumes) to avoid ENOSPC.
4. **Post-deploy validation.** Unauth smoke (site serves, redirects correct, new assets/routes live, removed routes 404) — needs no session. Authed smoke on the deployed stack catches deploy-layer issues (CDN/Lambda/RSC) the local preview can't: if the app is behind a real IdP, the dev-auth bypass is prod-disabled, so use a **captured session** (Crucible cookie-handoff state) — curl authed routes for tell-tale markers, or run Crucible regression. A feature rendering in prod that depends on a new column also *proves the migration applied*.
5. **Crucible** is the post-deploy prod regression guard for backend (b) projects. If the batch changed shared chrome, existing baselines will show **expected** drift (not regressions) and new surfaces have none — capture fresh baselines as a follow-up rather than blocking.

## Worktree conventions
- **Location:** `<repo>.worktrees/<branch-slug>` (sibling to main checkout). **Slug:** `/`→`-`. **One worktree per branch**, lifecycle create→work→merge→remove. Cleanup is idempotent (`prune` + fs scan).
- **Cross-repo batches:** a batch may touch a sibling repo (app + infra). Worktrees are per-repo; for the sibling, branch + PR in that repo directly. Deploy/CDK runs from the sibling's main, so its change must be merged there before you dogfood it.

## Failure handling summary
| Failure | Response |
|---|---|
| Feature agent crashes mid-implementation | Leave worktree, surface with retry/skip |
| Code review finds structural issue | Pause that PR, let wave continue, batch into review |
| Visual QA ISSUES_FOUND (localized) | Fix-forward via cleanup agent at the worktree, re-verify; don't merge a failing AC |
| QA NEEDS_HUMAN | Surface screenshots inline; quick human decision |
| Rebase conflict on wave-mate | Surface; do not auto-resolve (signals missed dep / scope overlap) |
| Regression finds real regression | Pause batch, do not start next wave |
| Prod migration path unknown | STOP — investigate, don't improvise prod DDL |
| Worktree cleanup fails | `--force` + fs delete + `prune` |

## Rules
- **Refinement is upstream.** Ambiguous story mid-run → stop and refine, don't guess.
- **QA agents own their environment.** They boot/tear down test-envs or preview servers; the orchestrator doesn't (except the one linchpin validation of a new dev-auth bypass).
- **Orchestrator approves baselines.** Never the QA agent.
- **Orchestrator does judgment, not line-edits.** Delegate fixes to cleanup agents.
- **Model + merge authority + QA backend are kickoff config**, not hardcoded.
- **No evidence on failed QA.** Only PASS posts to a PR.
- **Worktrees, not branch-juggling.** Never parallel agents on the main checkout.
- **Don't improvise prod.** Discover the real deploy/migration path; diff before every prod-mutating command; schema-first.
- **Project-agnostic.** No hardcoded counts/paths/env beyond the worktree convention. Project specifics live in that project's notes / QA backend.

## Iteration log

**v3 — 2026-06-01 — Promoted into the hl plugin + autri EPIC-5 lessons.** Moved from a standalone workspace skill into `hl/skills/ship` as the methodology's Ship stage. Validated on autri EPIC-5: **6 stories, 4 waves, one-shot** (every story correct on first agent pass; one localized fix-forward), then migrated + deployed to prod. New in v3:
- **QA backend is pluggable** — generalized 1d beyond the test-env+Crucible assumption to a second backend: **headless Claude Preview + a dev-only auth bypass** for apps with no per-branch harness that sit behind a real IdP. Sub-agents drive Preview (serially). The dev-auth bypass became **story-zero** (all visual QA depended on it); validated once as the linchpin, then its login recipe was reused by every wave's QA agent.
- **Merge authority is configurable** — added the delegated mode (orchestrator merges when QA is clean) alongside the default human-merge gate; set at kickoff.
- **Phase 2.5 deploy+migrate** — added an optional deploy phase. Headline lesson: **the migration runner may already exist, hidden in infra** (autri's was a CDK custom-resource Lambda, invisible from the app repo) — investigate before building, and never improvise prod DDL. Schema-first ordering. Built a unified `deploy.sh` CD dispatcher + auto-derived the migration trigger from the file count, then dogfooded both for the real deploy.
- **Main-green gate by the project's CI contract** — distinguish the real CI gate from out-of-CI integration suites; a red out-of-CI suite (unrelated dep bump) is flag-and-spawn, not a batch blocker.
- **Deps serialize same-file stories** — encode a dep to put two header-touching stories in different waves and dodge the guaranteed rebase conflict.
- **Model is kickoff config** — ran the whole batch on Opus for max quality (the human's call); the skill no longer hardcodes "Sonnet default."

**v2.1 — 2026-04-18 — Lessons from E12 W1+W2 (Foundry).** Phase 0 gates added after real incidents: main-green gate (main was red with stale assertions); deploy-env readiness gate (a `throw`-on-boot check took Foundry down — a required secret wasn't set on Fly); routing-layer-above-app check (OAuth routes mounted in the app but the proxy didn't forward them → prod 404s). Added post-merge smoke (1h) — local and prod smoke diverge across a CDN/proxy layer.

**v2 — 2026-04-17 — Batch/wave rewrite (Foundry E12).** From single-story/single-surface to topo-sorted waves, parallel worktrees, conditional QA, wave gates with rebase, batch review.

**v1 notes retained:** background the Docker boot and poll `/api/health` (don't tail build logs — burns tokens); NEEDS_HUMAN subjective classification beats pixel-match for compositing artifacts; Phase 0 main sync catches local-vs-origin drift that inflates diffs.
