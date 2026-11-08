# Board

Rendered 2026-11-08 11:25 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Fewer agent turns, fewer tokens](project-turn-savings.md) (0)

`turn-savings` · Age 6d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md) (3)

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 4d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Codex engine: the forced workspace-write sandbox fails where bubblewrap cannot create user namespaces** | [`codex-sandbox-mode`](design-codex-sandbox-mode.md) | darn · light | Darn-Write | neve | 6m<br>0m | codex, shuttle, preflight |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 2d<br>1d | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 5d<br>19h | shuttle, codex, loom |

## [P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes](project-fabric-test-speed-work.md) (0)

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 21h · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P0 Root: entries under no project](project-root.md) (22)

Owner `neve`

### Tasks (5)

Concrete and claimable; only a task can run.

#### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Run the cmd/lym tests in the Webster batch gate so verify failures surface in the fork**<br>• Batch gate also runs cmd/lym tests, so a break there is fixed by the batch fork instead of a verify-fix round | [`batch-gate-cmd-lym`](design-batch-gate-cmd-lym.md) | darn · light | Finalize | neve | 28m<br>28m | webster, gate |
| T2 | **Slack relay: one Socket Mode connection per machine, one channel per orch, two tokens** | [`slack-push-channels`](design-slack-push-channels.md)<br>[Sketch](decision-slack-push-channels.md) | tapestry · solid | Webster | neve | 1h<br>1h | slack, creel, orch |
| T3 | **Re-measure writer lookups and Plan phase cost on the tapestry runs since 3085f451**<br>• One read-only study: early lookup delegation and its carry, and whether the Plan phase cost has fallen since 2026-10-24 | [`writer-cost-remeasure`](design-writer-cost-remeasure.md) | darn · docs | Finalize | neve | 3d<br>26m | loom, stencil |
| T4 | **Beam reed session gets the task button group instead of beam** | [`beam-reed-task-buttons`](design-beam-reed-task-buttons.md) | darn · light | Darn-Write | neve | 6m<br>0m | reed, pawl |
| T5 | **Burler fix round: decision-class rule conflicts with parent ruling mail, fixer has no outlet to parent** | [`fixer-parent-ruling`](design-fixer-parent-ruling.md) | darn · light | Seed-Child | neve | 6m<br>0m | burler, stencil |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (17)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>2d | loom, stencil |

#### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 18h<br>18h | heddle |
| E3 |  | **Slack messages to the orch say Slack twice: sender ly:SLACK and a [slack] body prefix** | [`slack-forward-prefix`](design-slack-forward-prefix.md) | 2h<br>56m | slack |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 5d<br>2d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 7d<br>4d | shed, shuttle, batten |

#### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>12d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 2d<br>2d | reed, shuttle |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 2h<br>56m | slack, creel, orch |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 9d<br>2d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>3d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 5d<br>18h | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>2d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>18h | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 5d<br>2d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>2d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 3d<br>23h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 3d<br>3d | batten, board, fabric, workspaces |
