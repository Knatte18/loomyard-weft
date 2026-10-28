# Board

Rendered 2026-10-28 14:13 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Projects (2)

Groups of entries; a beam spawns its members from its own worktree.

### P1 Workspaces: beams, pairs from a parent pair, dobby

Beams that develop a project in their own worktree, driven by a dobby daemon

grouping

_None._

### P2 .NET: LoomYard for C# repositories

Running LoomYard on C#/.NET repositories such as Models: scoped gates, regression, a sandbox hub

grouping

| # | Member | Kind | Standing |
| --- | --- | --- | --- |
| E28 | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | Note | Note |

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **The watcher wakes the orch after a usage limit resets, and holds the repeat notices**<br>• `profile: solid` | [`usage-limit-wake`](design-usage-limit-wake.md) | darn | Code-Burler-Fix | neve | 54m<br>22m | orch, batten, shuttle |
| T2 | **A merge fixer change over the caps gets a solo review of its own commits, not a full Code review** | [`fixer-overcap-review`](design-fixer-overcap-review.md) | darn | Code-Burler-Fix | neve | 37m<br>22m | loom, burler |
| T3 | **Friction-Reflect producer fails to start with 'text file busy' when the lym binary is replaced mid-run** | [`spawn-text-file-busy`](design-spawn-text-file-busy.md) | darn | Code-Bouncer | neve | 54m<br>22m | shed, deploy, loom |
| T6 | **The dobby holds a new run start while the user slice nears its task limit**<br>• Hold new run starts when the user slice pids.current nears pids.max | [`dobby-task-limit-hold`](design-dobby-task-limit-hold.md) | darn | Finalize | neve | 2h<br>22m | batten |
| T7 | **Watcher's /compact inject into an idle orch is typed but not submitted (unconfirmed, then stalled on draft)** | [`compact-inject-draft`](design-compact-inject-draft.md) | darn | Darn-Write | neve | 23m<br>22m | creel, shuttle, orch |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Shuttle reads activity and operator input from Claude Code hooks**<br>• PostToolUse activity for the idle reader, UserPromptSubmit carried over on respawn, subagent and compact signals<br>• `profile: solid` | [`shuttle-hook-signals`](design-shuttle-hook-signals.md) | darn | `usage-limit-wake`, `compact-inject-draft` | 4d<br>22m | shuttle, batten, shed, reed, claude |

## Notes (31)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (14)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Beam promote keeps the Discussion record as part of the beam**<br>• When Discussion finds a task should be a project, promote carries its decision record into the beam instead of dropping it | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 3d<br>1d | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>1h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 23h<br>20h | batten, stencil, config |
| E5 |  | **Decide the layout of projects in the board README**<br>• The operator is not satisfied with how the README renders projects and their members; the layout is to be decided | [`board-projects-layout`](design-board-projects-layout.md) | 3d<br>23h | board |
| E6 |  | **Board README: one rendered file per project, research notes first**<br>• Each project renders its own file linked from its parent; README.md is the root project; research notes come right after tasks | [`readme-per-project-files`](design-readme-per-project-files.md) | 2d<br>22h | board |
| E7 |  | **Run verbs in one shared namespace, not under lym loom**<br>• lym loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 13d<br>1d | shed, loom |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 2w<br>1d | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 13d<br>13d | board, loom |
| E14 |  | **No test is skipped by design**<br>• Windows-only tests count as skips on Linux, and TestBuildLoomShed_OutputShape skips on every machine | [`test-skips-os-tags`](design-test-skips-os-tags.md) | 2d<br>2d | testkit, reed, loom, fabric, gate |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 10d<br>1d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 9d<br>1d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 5d<br>3h | burler, claude |
| E28 |  | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 6d<br>1d | dotnet |
| E32 |  | **Move rules about lym behaviour out of this repo's CLAUDE.md into lym code, doc.go and stencils** | [`claude-md-lym-rules-out`](design-claude-md-lym-rules-out.md) | 6d<br>1d | stencil, board, loom |

### Features (12)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>1d | burler |
| F2 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lym-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 3w<br>1d | ide, fabric |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>1d | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>1d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>4h | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 13d<br>1d | reed, config, batten |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 13d<br>1h | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 12d<br>1d | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 5d<br>1d | loom, shed, batten, workspaces, burler |

### Researchs (5)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 10d<br>2d |  |
| R3 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 8d<br>2d |  |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 6d<br>1d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 5d<br>3h | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 1d<br>1d | stencil, loom, burler |
