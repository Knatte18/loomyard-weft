# Board

Rendered 2026-10-28 10:20 UTC; every age below is as of then.
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

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Creel delivers every post to a Claude strand through hooks, visibly and in full**<br>• PostToolUse, Stop and asyncRewake hooks carry posts as fast as possible; shuttle appends a trust and reply sentence to the system prompt | [`creel-hook-post`](design-creel-hook-post.md) | loom | Plan-Write | neve | 5d<br>1h | creel, shuttle, claude |
| T2 | **A landing merges a moved parent in Finalize and sends a fixer escalate to the arbiter first**<br>• No Finalize reroute to Parent-Align for a moved parent, and a landing fixer escalate reaches the orch only as the arbiter hand-on | [`landing-settles-in-place`](design-landing-settles-in-place.md) | loom | Plan-Burler-Review | neve | 4d<br>50m | loom, batten, shed |
| T4 | **One verb that pulls the whole hub, Bolt included, while the orch runs**<br>• Pull the prime pair, Bolt and local beams in one safe verb | [`hub-pull-verb`](design-hub-pull-verb.md) | darn | Code-Burler-Review | neve | 1d<br>50m | fabric, board |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (33)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs (2)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Find why a running run shows a dead waiter in the activity reader** | [`idle-notice-dead-waiter-cause`](design-idle-notice-dead-waiter-cause.md) | 4d<br>4d | batten, shuttle |
| B7 |  | **An instruction typed into a pane is lost when the tackler re-spawns the row**<br>• A re-spawned row session starts from its prompt alone; operator input to the old session is dropped | [`pane-input-lost-on-respawn`](design-pane-input-lost-on-respawn.md) | 1d<br>1d | shed, shuttle, reed |

### Enhancements (14)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Beam promote keeps the Discussion record as part of the beam**<br>• When Discussion finds a task should be a project, promote carries its decision record into the beam instead of dropping it | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 2d<br>1d | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>19h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 19h<br>16h | batten, stencil, config |
| E5 |  | **Decide the layout of projects in the board README**<br>• The operator is not satisfied with how the README renders projects and their members; the layout is to be decided | [`board-projects-layout`](design-board-projects-layout.md) | 3d<br>19h | board |
| E6 |  | **Board README: one rendered file per project, research notes first**<br>• Each project renders its own file linked from its parent; README.md is the root project; research notes come right after tasks | [`readme-per-project-files`](design-readme-per-project-files.md) | 2d<br>19h | board |
| E7 |  | **Run verbs in one shared namespace, not under lym loom**<br>• lym loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 13d<br>1d | shed, loom |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 2w<br>1d | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 12d<br>12d | board, loom |
| E14 |  | **No test is skipped by design**<br>• Windows-only tests count as skips on Linux, and TestBuildLoomShed_OutputShape skips on every machine | [`test-skips-os-tags`](design-test-skips-os-tags.md) | 2d<br>2d | testkit, reed, loom, fabric, gate |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 10d<br>1d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 9d<br>1d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 5d<br>0m | burler, claude |
| E28 |  | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 6d<br>1d | dotnet |
| E32 |  | **Move rules about lym behaviour out of this repo's CLAUDE.md into lym code, doc.go and stencils** | [`claude-md-lym-rules-out`](design-claude-md-lym-rules-out.md) | 5d<br>1d | stencil, board, loom |

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
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>53m | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 13d<br>1d | reed, config, batten |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 13d<br>0m | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 12d<br>1d | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 5d<br>1d | loom, shed, batten, workspaces, burler |

### Researchs (5)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 9d<br>2d |  |
| R3 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 8d<br>2d |  |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 5d<br>1d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 5d<br>0m | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 1d<br>1d | stencil, loom, burler |
