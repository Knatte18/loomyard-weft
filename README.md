# Board

Rendered 2026-10-29 08:53 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Projects (2)

- [P1 Workspaces: beams, pairs from a parent pair, dobby](project-workspaces.md) — Beams that develop a project in their own worktree, driven by a dobby daemon
- [P2 .NET: LoomYard for C# repositories](project-dotnet.md) — Running LoomYard on C#/.NET repositories such as Models: scoped gates, regression, a sandbox hub

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Fix two flaky creel tests and name failing tests in the adopt mail**<br>• Make the daemon lifecycle and TestCreelCLI EXTERNAL tests deterministic, and list failed tests in the adopt check mail.<br>• `profile: light` | [`creel-flakes-and-adopt-names`](design-creel-flakes-and-adopt-names.md) | darn | Code-Burler-Fix | osl-1046 | 1h<br>37m | creel, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info<br>• `profile: docs` | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | darn | T3 | 9d<br>5m |  |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Act on the first test-time audit: reedengine wait, three over-grown test packages**<br>• Injectable wait for reedengine tier2 tests (40 s); prune tests in shell, boardtest and battenrecipe | [`test-prune-audit-leads`](design-test-prune-audit-leads.md) | darn |  | 13h<br>13h | reed, board, batten |

## Notes (33)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 6d<br>1d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 6d<br>22h | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 2d<br>15h | stencil, loom, burler |

### Bugs (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B3 |  | **Flaky websterengine recover-batch subtest fails once with "record changed"** | [`webster-recover-batch-flake`](design-webster-recover-batch-flake.md) | 1h<br>1h | webster |
| B5 |  | **Burler fork guard halts a run on a read-only git command** | [`burler-guard-readonly-git`](design-burler-guard-readonly-git.md) | 1h<br>1h | burler, shed |
| B6 |  | **Card gate runs tier0 only and misses tier1 breakage a card causes** | [`card-gate-misses-tier1`](design-card-gate-misses-tier1.md) | 1h<br>1h | webster, gate |

### Enhancements (14)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Beam promote keeps the Discussion record as part of the beam**<br>• When Discussion finds a task should be a project, promote carries its decision record into the beam instead of dropping it | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 3d<br>1d | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>15h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 1d<br>1d | batten, stencil, config |
| E4 |  | **Restore the verify-gate test for a misplaced //lym:guard marker**<br>• Once the darn verify key reads its diff base, cover the guard scan of a narrowed run again | [`verifygate-misplaced-guard-test`](design-verifygate-misplaced-guard-test.md) | 17h<br>17h | loom, gate |
| E5 |  | **The idle ring hands the mail over itself, without an empty turn**<br>• hook-wait wakes an idle strand with a bell line only, so each mail costs one extra empty turn | [`creel-ring-carries-mail`](design-creel-ring-carries-mail.md) | 14h<br>14h | creel |
| E6 |  | **A module label for every package group under internal/**<br>• Several lym modules have no label, so entries take the nearest one that says something else | [`label-coverage`](design-label-coverage.md) | 14h<br>14h | board |
| E7 |  | **loom names the general run; today's loom is one variant**<br>• loom becomes the general driver of a Shed run, today's loom, darn and Hardener its variants, and lym loom stays as it is. | [`run-verbs-shared`](design-run-verbs-shared.md) | 2w<br>0m | shed, loom |
| E8 |  | **lym ide seed colours the VS Code workspace too**<br>• A hub opened as a VS Code workspace has no colour, since lym ide seed seeds only worktrees. | [`workspace-ide-seed`](design-workspace-ide-seed.md) | 1h<br>1h | ide, workspaces |
| E9 |  | **Reed navigation keys, Alt+arrows, are seeded by lym**<br>• Alt+arrow keys do not switch windows and sessions in Reed in this hub; lym should seed them. | [`reed-alt-arrow-keys-seeded`](design-reed-alt-arrow-keys-seeded.md) | 1h<br>1h | reed, config |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 13d<br>13d | board, loom |
| E11 |  | **Creel daemon writes no delivery-level log, so an unconfirmed delivery cannot be diagnosed** | [`creel-delivery-log`](design-creel-delivery-log.md) | 1h<br>1h | creel |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 11d<br>1d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 10d<br>1d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 6d<br>18h | burler, claude |

### Features (13)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>1d | burler |
| F2 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lym-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 3w<br>1d | ide, fabric |
| F3 |  | **Windows lym opens land tiled on the right KDE virtual desktop**<br>• Konsole and VS Code windows opened by lym placed by desktop and tile on KDE Plasma | [`kde-window-placement`](design-kde-window-placement.md) | 17h<br>17h | reed, ide, config |
| F4 |  | **A landing notice on request, per task**<br>• The orch can ask for one creel line when a named run lands; off unless set for that slug | [`landing-watch`](design-landing-watch.md) | 14h<br>14h | dobby, creel, orch |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>15h | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>1d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>23h | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 2w<br>1d | reed, config, batten |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 13d<br>17h | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 6d<br>1d | loom, shed, batten, workspaces, burler |
