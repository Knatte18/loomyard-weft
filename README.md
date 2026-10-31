# Board

Rendered 2026-10-31 15:04 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Prime

The root project: the entries under no other project.

### Tasks (6)

Concrete and claimable; only a task can run.

#### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **A landing notice on request, per task**<br>• A landing notice from dobby on request, per task, reusing the wait that lym hibernate request --after has<br>• `profile: solid` | [`landing-watch`](design-landing-watch.md) | darn | Darn-Write | neve | 2d<br>23m | dobby, creel, orch, board, reed, config |
| T4 | **loom becomes the task-run contract that tapestry and darn fulfil**<br>• loom becomes the general driver of a Shed run, today's loom, darn and Hardener its variants, and lym loom stays as it is. | [`run-verbs-shared`](design-run-verbs-shared.md) | tapestry | Discussion-Burler-Fix | neve | 2w<br>1h | shed, loom, darn, landing, drift |
| T5 | **Second recurring test-time and growth audit**<br>• Re-time tier0 to tier2 by the first audit's method and append the section<br>• `profile: docs` | [`test-time-audit-2`](design-test-time-audit-2.md) | darn | Finalize | neve | 48m<br>22m | gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Landing commits drop Co-Authored-By and Lym-Task, and carry an optional credit listing the engines used**<br>• Replace the per-engine Co-Authored-By line and the Lym-Task line with one optional credit block that lists the LLM services the task used. | [`commit-signature-credit`](design-commit-signature-credit.md) | darn | T4 | 1d<br>43m | landing |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Pawl becomes a loom recipe, with a project decision record and no review rounds**<br>• Pawl joins the loom contract; a project record written at beam create and changed only through decision add<br>• `profile: solid` | [`pawl-loom-recipe`](design-pawl-loom-recipe.md) | darn | T4<br>T6<br>T3 | 1h<br>22m | pawl, loom, board, workspaces |

#### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **sampler: an interactive loom recipe, the operator iterating with an agent in the run's worktree until done**<br>• A Shed recipe whose agent waits for the operator and iterates on the result, then lands as usual | [`interactive-recipe`](design-interactive-recipe.md) | tapestry | T4<br>T1 | 2h<br>22m | shed, batten, reed, shuttle |

### Notes (31)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 9d<br>4d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 8d<br>3d | loom, stencil |

#### Enhancements (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A task that turns out to be a project is refiled as a project, keeping its Discussion findings**<br>• The agent files a note with its findings and tells the orch; the operator decides, then the run ends and a new beam spawns from the parent | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 6d<br>1d | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>2h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 3d<br>3d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 2d<br>2d | dotnet, gate |
| E5 |  | **pr-continue wakes the PR-Review wait at once**<br>• The PR-Review wait sleeps up to a minute between wakes, so lym loom pr-continue takes effect up to a minute late | [`pr-continue-wakes-row`](design-pr-continue-wakes-row.md) | 1d<br>1d | landing, batten |
| E6 |  | **Gate runs spend much of their time starting processes: start fewer**<br>• launch_seconds is a large share of a gate run on OSL-1046, where each process start is slow; the aim is a faster gate by starting fewer processes | [`gate-fewer-launches`](design-gate-fewer-launches.md) | 1d<br>1d | gate |
| E7 |  | **Finalize settles a rejected fixer change in place and never sends the run back**<br>• A fix review loop inside Finalize that escalates at its cap, in place of on_reroute | [`finalize-fix-in-place`](design-finalize-fix-in-place.md) | 2h<br>2h | landing, shed, loom, darn |
| E8 |  | **A seed says only that a run starts at its start, with no producer name**<br>• The shared seed names Preflight as the first producer for every recipe; the seed should name none, and the recipe supplies its first row. | [`seed-starts-at-start`](design-seed-starts-at-start.md) | 1d<br>1d | loom, pawl, shed |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 30m<br>30m | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby, once dobby and hibernate share the past-Land test | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 24m<br>24m | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, in place of pawl's perch round**<br>• A ready-made command starts a run that brings the parent into a beam, with Drift; pawl drops its own Code-Review perch | [`parent-merge-task`](design-parent-merge-task.md) | 2h<br>2h | pawl, workspaces, drift, landing |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 13d<br>4d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 12d<br>4d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 8d<br>3d | burler, claude |

#### Features (15)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>4d | burler |
| F2 | H | **warp-visibility: LoomYard adds no instruction file to a repository**<br>• No CLAUDE.md or CLAUDE.local.md of lym in any repo; open: where a repo own rules for lym agents live out of sight of its team | [`warp-visibility`](design-warp-visibility.md) | 3w<br>2d | ide, fabric |
| F3 |  | **Windows lym opens land tiled on the right KDE virtual desktop**<br>• Konsole and VS Code windows opened by lym placed by desktop and tile on KDE Plasma | [`kde-window-placement`](design-kde-window-placement.md) | 2d<br>2d | reed, ide, config |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 1d<br>1d | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 1d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **One verb to take work over on another machine, leaving every run in a known state**<br>• Moving to another machine is many hand steps, and afterwards run states are unclear | [`machine-switch`](design-machine-switch.md) | 3h<br>3h | fabric, dobby, batten |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>1d | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 3w<br>4d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>3d | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 2w<br>14m | reed, config, batten |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>2d | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 8d<br>4d | loom, shed, batten, workspaces, burler |
