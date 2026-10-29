# Board

Rendered 2026-10-29 13:35 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Creel mail reaches the model and the operator once, in one message, and is logged**<br>• Fix the doubled Stop-hook delivery and its error label, let the idle ring carry the mail, and log each delivery attempt<br>• `profile: solid` | [`creel-mail-delivered-once`](design-creel-mail-delivered-once.md) | loom | Plan-Bouncer | osl-1046 | 19h<br>1h | creel |
| T2 | **PR-Review opens a pull request and starts an interactive agent to discuss it**<br>• Build the PR-Review stub: a pull request per parent branch setting, a talkable agent on its own configured model, and a landing that follows the pull request | [`pr-review-row`](design-pr-review-row.md) | loom | Plan-Write | osl-1046 | 1h<br>37m | loom, shed, batten, shuttle, config |
| T3 | **Burler fork guard halts a run on a read-only git command** | [`burler-guard-readonly-git`](design-burler-guard-readonly-git.md) | darn | Describe | osl-1046 | 5h<br>23m | burler, shed |
| T6 | **Card gate runs tier0 only and misses tier1 breakage a card causes** | [`card-gate-misses-tier1`](design-card-gate-misses-tier1.md) | darn | Darn-Write | osl-1046 | 5h<br>20m | webster, gate |
| T7 | **PATTERN entries and scans for interactive spawns and filesystem links**<br>• Move the tmux-only spawn rule and the fslink rule from CLAUDE.md into PATTERN with a scan each | [`pattern-spawn-and-links-scans`](design-pattern-spawn-and-links-scans.md) | darn | Finalize | osl-1046 | 16m<br>16m | pattern, shuttle, fabric |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Delete the loomyard CLAUDE.md after moving any rule only it holds**<br>• Check each rule against PATTERN, stencils and skills, move what is missing, guard update-plugins.sh, then delete CLAUDE.md | [`claude-md-lym-rules-out`](design-claude-md-lym-rules-out.md) | darn | T7 | 1h<br>16m | stencil, pattern, orch |

## Notes (28)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 6d<br>2d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 6d<br>1d | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 2d<br>20h | stencil, loom, burler |

### Bugs (1)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Stop-hook creel mail arrives twice and is labelled as an error**<br>• One creel mail shows twice at the Stop hook, once as feedback and once as Stop hook error, with the hook command text in the model context | [`stop-hook-mail-doubled`](design-stop-hook-mail-doubled.md) | 12m<br>11m | creel |

### Enhancements (11)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Beam promote keeps the Discussion record as part of the beam**<br>• When Discussion finds a task should be a project, promote carries its decision record into the beam instead of dropping it | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 4d<br>2d | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>20h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 1d<br>1d | batten, stencil, config |
| E6 |  | **A module label for every package group under internal/**<br>• Several lym modules have no label, so entries take the nearest one that says something else | [`label-coverage`](design-label-coverage.md) | 18h<br>10m | board |
| E7 |  | **loom names the general run; today's loom is one variant**<br>• loom becomes the general driver of a Shed run, today's loom, darn and Hardener its variants, and lym loom stays as it is. | [`run-verbs-shared`](design-run-verbs-shared.md) | 2w<br>4h | shed, loom |
| E8 |  | **Dobby sends the batten notices, not a row of the batten recipe**<br>• Move quiet, driver-dead and api-error detection from battenshed/notice.go into dobby loop over the runs it drives | [`notices-from-dobby`](design-notices-from-dobby.md) | 27m<br>27m | dobby, batten |
| E12 |  | **Revise the board README rendering**<br>• The README hides project members behind a bare link and has other unclear spots; the operator wants the rendering revised. | [`readme-omits-csharp-note`](design-readme-omits-csharp-note.md) | 3h<br>10m | board |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 11d<br>2d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 10d<br>2d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 6d<br>23h | burler, claude |
| E28 |  | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 7d<br>1h | dotnet |

### Features (13)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>2d | burler |
| F2 | H | **warp-visibility: LoomYard adds no instruction file to a repository**<br>• No CLAUDE.md or CLAUDE.local.md of lym in any repo; open: where a repo own rules for lym agents live out of sight of its team | [`warp-visibility`](design-warp-visibility.md) | 3w<br>1h | ide, fabric |
| F3 |  | **Windows lym opens land tiled on the right KDE virtual desktop**<br>• Konsole and VS Code windows opened by lym placed by desktop and tile on KDE Plasma | [`kde-window-placement`](design-kde-window-placement.md) | 22h<br>22h | reed, ide, config |
| F4 |  | **A landing notice on request, per task**<br>• The orch can ask for one creel line when a named run lands; off unless set for that slug | [`landing-watch`](design-landing-watch.md) | 18h<br>18h | dobby, creel, orch |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>20h | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>2d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>1d | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 2w<br>2d | reed, config, batten |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 13d<br>22h | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 6d<br>2d | loom, shed, batten, workspaces, burler |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T4 | **Restore the verify-gate test for a misplaced //lym:guard marker**<br>• Once the darn verify key reads its diff base, cover the guard scan of a narrowed run again | [`verifygate-misplaced-guard-test`](design-verifygate-misplaced-guard-test.md) | 22h<br>23m | loom, gate |
