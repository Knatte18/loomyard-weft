# Board

Rendered 2026-11-01 07:28 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Prime

The root project: the entries under no other project.

### Tasks (4)

Concrete and claimable; only a task can run.

#### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Dobby sends a dead-driver notice for a run that has just started**<br>• Dobby reads a go-driven child as dead in the gap between loom start writing its running status and the driver taking its lock<br>• `profile: solid` | [`dead-driver-false-notice`](design-dead-driver-false-notice.md) | darn | Code-Burler-Review | neve | 1h<br>1h | dobby, loom |
| T2 | **warp-visibility: heddle, a Bolt database lym renders its instruction files from, leaving no trace in warp**<br>• LoomYard puts no CLAUDE.md in a repository; lym renders its own git-excluded instruction files into each worktree from _board/heddle.json | [`warp-visibility`](design-warp-visibility.md) | tapestry | Discussion-Burler-Review | neve | 3w<br>1h | fabric, board, config, claude, codex, cwd, process, stencil, shuttle |
| T6 | **Finalize settles a rejected fixer change in place and never sends the run back**<br>• A fix review loop inside Finalize that escalates at its cap, in place of on_reroute<br>• `profile: solid` | [`finalize-fix-in-place`](design-finalize-fix-in-place.md) | darn | Code-Burler-Fix | neve | 18h<br>1h | landing, shed, loom, darn |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Rename the Bolt checkout directory _board to _bolt, by script alone**<br>• A pure, mechanical rename of _board to _bolt everywhere, with a rename program for existing hubs | [`bolt-dir-rename`](design-bolt-dir-rename.md) | darn | T2<br>T6<br>T1 | 15h<br>1h | fabric, board, config, cwd, stencil |

### Notes (31)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 | H | **Trial the cuts that make a gate run link fewer new test binaries**<br>• Bench the report's cuts on Neve and OSL-1046 before any is made permanent | [`gate-fewer-binaries-trial`](design-gate-fewer-binaries-trial.md) | 13h<br>13h | gate, testkit |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 9d<br>4d | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 9d<br>29m | loom, stencil |

#### Enhancements (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A task that turns out to be a project is refiled as a project, keeping its Discussion findings**<br>• The agent files a note with its findings and tells the orch; the operator decides, then the run ends and a new beam spawns from the parent | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 6d<br>16h | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>18h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 4d<br>4d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 2d<br>2d | dotnet, gate |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 16h<br>29m | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 16h<br>12h | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A ready-made command starts a run that brings the parent into a beam, with Drift; pawl drops its own Code-Review perch | [`parent-merge-task`](design-parent-merge-task.md) | 18h<br>0m | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 14h<br>14h | fabric |
| E13 |  | **A long gate run answers within the cache lifetime, so the waiting agent keeps its prompt cache warm**<br>• lym gate test caps the run wait like the slot wait, with the cap a config value beside the engine cache lifetime | [`gate-run-keeps-cache-warm`](design-gate-run-keeps-cache-warm.md) | 13h<br>13h | gate, shuttle, config |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 2w<br>4d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 13d<br>4d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 9d<br>3d | burler, claude |

#### Features (16)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>4d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 12h<br>12h | reed, board, shuttle, orch |
| F3 |  | **A window lym opens is snapped into the active desktop tiling by a configured command**<br>• After opening a window lym runs a machine-local configured command, such as the operator tiling script; lym knows nothing about KDE | [`kde-window-placement`](design-kde-window-placement.md) | 3d<br>15h | reed, ide, config |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 12h<br>12h | orch, dobby, arbiter |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 4w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 1d<br>1d | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 1d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **One verb to take work over on another machine, leaving every run in a known state**<br>• Moving to another machine is many hand steps, and afterwards run states are unclear | [`machine-switch`](design-machine-switch.md) | 19h<br>19h | fabric, dobby, batten |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>1d | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 3w<br>4d | reed, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>3d | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 2w<br>16h | reed, config, batten |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>3d | burler, shed, orch, quarry, report |
| F20 |  | **stitch: a loom recipe that splits a large task into darn runs inside its own beam**<br>• Discussion, a pre-planner, the pieces filed as darn tasks in the task's project, and an inner pawl run that builds and lands the beam | [`loom-beam-darns`](design-loom-beam-darns.md) | 9d<br>10h | loom, shed, batten, workspaces, burler |
