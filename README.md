# Board

Rendered 2026-11-08 18:08 UTC; every age below is as of then.
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

## [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md) (2)

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 4d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 2d<br>5h | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 5d<br>6h | shuttle, codex, loom |

## [P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes](project-fabric-test-speed-work.md) (1)

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 1d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Second fabricengine test cut, from the coverage study's removable set** | [`fabric-test-cuts-2`](design-fabric-test-cuts-2.md) | darn · solid | Seed-Child | neve | 23m<br>0m | fabric, testkit |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P4 Arras: one recipe that runs Motif and raddle in a row, with a perch on the split](project-arras-recipe.md) (5)

[`arras-recipe`](design-arras-recipe.md) · Age 1h · Owner `neve`

### Tasks (5)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Releasing a project with lym board set-status releases its draft member tasks** | [`project-release-members`](design-project-release-members.md)<br>[Sketch](decision-project-release-members.md) | darn · light |  | 1h<br>1h | shed, loom, burler, arbiter, board |
| T5 | `draft` **Raddle's perch mode: recorded from the recipe's rows, with its outcome checks and refusals** | [`raddle-perch-mode`](design-raddle-perch-mode.md)<br>[Sketch](decision-raddle-perch-mode.md) | darn · solid |  | 1h<br>1h | shed, loom, burler, arbiter, board |
| T6 | `draft` **The arras recipe and its split perch** | [`arras-recipe-perch`](design-arras-recipe-perch.md)<br>[Sketch](decision-arras-recipe-perch.md) | darn · solid | T5 | 1h<br>1h | shed, loom, burler, arbiter, board |
| T7 | `draft` **An arras family filing turns the run's entry into its project and re-points its dependents** | [`arras-family-filing`](design-arras-family-filing.md)<br>[Sketch](decision-arras-family-filing.md) | darn · solid | T4<br>T5<br>T6 | 1h<br>1h | shed, loom, burler, arbiter, board |
| T8 | `draft` **The shipped rule and docs name arras as the recipe for a tapestry-sized note** | [`arras-shipped-text`](design-arras-shipped-text.md)<br>[Sketch](decision-arras-shipped-text.md) | darn · docs | T4<br>T5<br>T6<br>T7 | 1h<br>1h | shed, loom, burler, arbiter, board |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P0 Root: entries under no project](project-root.md) (19)

Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **A mechanical way to find redundant tests: package-scoped coverage, a map per package, a check on a diff** | [`redundant-test-finder`](design-redundant-test-finder.md) | motif | Discussion-Burler-Fix | neve | 56m<br>50m | testkit, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (18)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>5h | loom, stencil |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B2 |  | **A loop that ends without an envelope records no exit status, so its halt cannot be explained** | [`loop-exit-not-recorded`](design-loop-exit-not-recorded.md) | 2h<br>2h | shed, friction |

#### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 1d<br>1d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 4h<br>4h | shed, loom, darn |
| E3 |  | **lym gate test runs a slow package as several processes, each taking a share of its tests** | [`gate-shard-slow-package`](design-gate-shard-slow-package.md) | 1h<br>38m | gate |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 8d<br>5d | shed, shuttle, batten |

#### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>12d | burler |
| F2 |  | **One desktop shortcut brings every lym session of the machine back after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 3h<br>3h | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 8h<br>5h | slack, creel, orch |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 2h<br>38m | loom, orch, config |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 9d<br>3d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>4d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 6d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>3d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>3d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 3d<br>1d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 3d<br>3d | batten, board, fabric, workspaces |
