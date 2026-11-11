# Board

Rendered 2026-11-11 19:29 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (3)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 8d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 5d<br>2d | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 2d<br>11h | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 8d<br>3d | shuttle, codex, loom |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (4)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **An upkeep pass runs under a slot it yields to a gate, side by side where tests declare it, and writes per round** | [`testmap-upkeep-runs`](design-testmap-upkeep-runs.md)<br>[Sketch](decision-testmap-upkeep-runs.md) | darn · solid | Darn-Write | neve | 2h<br>2h | testkit, gate, dobby |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **The prime dobby starts an upkeep pass on remaining work, and an anchor move ends it** | [`testmap-upkeep-dobby`](design-testmap-upkeep-dobby.md)<br>[Sketch](decision-testmap-upkeep-dobby.md) | darn · solid | T4 | 2h<br>2h | testkit, gate, dobby |

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 2d<br>4h | testkit, gate |
| E12 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 1d<br>4h | dobby, gate, testkit |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (20)</strong></summary>

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **The parent-review ask record, its deadline keys, and lym loom parent-review answer and await** | [`parent-review-ask-verbs`](design-parent-review-ask-verbs.md)<br>[Sketch](decision-parent-review-ask-verbs.md) | darn · solid |  | 10m<br>10m | loom, orch, config, burler |

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, the simple design: a fresh session picks by weight and headroom, a resume finds its transcript**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif |  | 9d<br>0m | claude, shuttle, config |
| T7 | `draft` **The Discussion-Review round asks the parent orch, and its chair keeps every parent finding** | [`discussion-review-parent-seat`](design-discussion-review-parent-seat.md)<br>[Sketch](decision-discussion-review-parent-seat.md) | darn · solid | T6 | 10m<br>10m | loom, orch, config, burler |

### Notes (17)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2d<br>2d | gate, fabric |
| R5 |  | **Bench a Sonnet darn writer with an Opus advisor on three landed darn tasks** | [`advisor-model-bench`](design-advisor-model-bench.md) | 51m<br>46m | darn, loom, shuttle, claude |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>2d | loom, stencil |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **The orch session starts on the provider default model because orch.model and orch.effort default to empty** | [`orch-model-default-empty`](design-orch-model-default-empty.md) | 47m<br>47m | orch, config |

#### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 4d<br>4d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 3d<br>40m | shed, loom, darn |
| E3 |  | **Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by** | [`selector-non-body-miss`](design-selector-non-body-miss.md) | 4h<br>46m | testkit, gate |

#### Features (10)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 3d<br>46m | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 3d<br>3h | slack, creel, orch |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 13h<br>11h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 12d<br>6d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>7d | board, ide, orch |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>6d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1h | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>6d | burler, shed, orch, quarry, report |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 6d<br>6d | batten, board, fabric, workspaces |

</details>
