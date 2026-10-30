# Board

Rendered 2026-10-30 12:35 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Revise the board README rendering**<br>• The README hides project members behind a bare link and has other unclear spots; the operator wants the rendering revised.<br>• `profile: light` | [`readme-omits-csharp-note`](design-readme-omits-csharp-note.md) | darn | Drift-Verify | osl-1046 | 1d<br>16m | board |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (32)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 7d<br>3d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 7d<br>2d | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 3d<br>5h | stencil, loom, burler |

### Bugs (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Stop-hook creel mail arrives twice and is labelled as an error**<br>• One creel mail shows twice at the Stop hook, once as feedback and once as Stop hook error, with the hook command text in the model context | [`stop-hook-mail-doubled`](design-stop-hook-mail-doubled.md) | 23h<br>5h | creel |
| B2 |  | **Dobby never starts a task whose slug has an old finished run record, and status gives no reason**<br>• A ready task sat unstarted for over an hour: batten refused its slug every pass, and beam status read ready to start | [`dobby-start-blocked-silently`](design-dobby-start-blocked-silently.md) | 22h<br>22h | dobby, batten |
| B3 |  | **Dobby's notice for a start that keeps failing reached the orch over an hour late**<br>• Dobby failed to start a task every pass from 13:43Z, but its mail reached the orch only an hour later | [`dobby-start-notice-late`](design-dobby-start-notice-late.md) | 22h<br>22h | dobby, creel |

### Enhancements (11)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A task that turns out to be a project is refiled as a project, keeping its Discussion findings**<br>• The agent files a note with its findings and tells the orch; the operator decides, then the run ends and a new beam spawns from the parent | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 4d<br>5h | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>1d | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 2d<br>2d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 22h<br>22h | dotnet, gate |
| E5 |  | **pr-continue wakes the PR-Review wait at once**<br>• The PR-Review wait sleeps up to a minute between wakes, so lym loom pr-continue takes effect up to a minute late | [`pr-continue-wakes-row`](design-pr-continue-wakes-row.md) | 10m<br>10m | landing, batten |
| E6 |  | **Gate runs spend much of their time starting processes: start fewer**<br>• launch_seconds is a large share of a gate run on OSL-1046, where each process start is slow; the aim is a faster gate by starting fewer processes | [`gate-fewer-launches`](design-gate-fewer-launches.md) | 2m<br>0m | gate |
| E7 |  | **loom names the general run; today's loom is one variant**<br>• loom becomes the general driver of a Shed run, today's loom, darn and Hardener its variants, and lym loom stays as it is. | [`run-verbs-shared`](design-run-verbs-shared.md) | 2w<br>1d | shed, loom |
| E8 |  | **Dobby sends the batten notices, not a row of the batten recipe**<br>• Move quiet, driver-dead and api-error detection from battenshed/notice.go into dobby loop over the runs it drives | [`notices-from-dobby`](design-notices-from-dobby.md) | 23h<br>23h | dobby, batten |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 12d<br>3d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 11d<br>3d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 7d<br>1d | burler, claude |

### Features (15)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>3d | burler |
| F2 | H | **warp-visibility: LoomYard adds no instruction file to a repository**<br>• No CLAUDE.md or CLAUDE.local.md of lym in any repo; open: where a repo own rules for lym agents live out of sight of its team | [`warp-visibility`](design-warp-visibility.md) | 3w<br>1d | ide, fabric |
| F3 |  | **Windows lym opens land tiled on the right KDE virtual desktop**<br>• Konsole and VS Code windows opened by lym placed by desktop and tile on KDE Plasma | [`kde-window-placement`](design-kde-window-placement.md) | 1d<br>22h | reed, ide, config |
| F4 |  | **A landing notice on request, per task**<br>• A landing notice from dobby on request, per task, reusing the wait that lym hibernate request --after has | [`landing-watch`](design-landing-watch.md) | 1d<br>1h | dobby, creel, orch |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 3h<br>3h | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2h<br>2h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>22h | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 3w<br>3d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>2d | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 2w<br>5h | reed, config, batten |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>1d | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 7d<br>3d | loom, shed, batten, workspaces, burler |
