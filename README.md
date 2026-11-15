# Board

Rendered 2026-11-15 16:33 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-adopt-revision.md">P1 Revise adopt and the functions that move a hub between machines</a> (1)</strong></summary>

[`adopt-revision`](design-adopt-revision.md) · Age 4h · Owner `neve`

### [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (9)

[`adopt-build`](design-adopt-build.md) · Age 35m · Owner `neve`

#### Tasks (9)

Concrete and claimable; only a task can run.

##### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder** | [`orch-resume-own-session`](design-orch-resume-own-session.md)<br>[Sketch](decision-orch-resume-own-session.md) | darn · solid | Code-Burler-Fix | neve | 36m<br>35m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T6 | **One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked** | [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)<br>[Sketch](decision-run-fsm-tracked-records.md) | tapestry · solid | Plan-Burler-Review | neve | 36m<br>35m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Draft (7)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay** | [`push-before-act-takeover`](design-push-before-act-takeover.md)<br>[Sketch](decision-push-before-act-takeover.md) | tapestry · solid | T6<br>T14 | 36m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T9 | `draft` **A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence** | [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md)<br>[Sketch](decision-bolt-holds-catch-up.md) | tapestry · solid | T8<br>T3<br>T14 | 36m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T10 | `draft` **lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs** | [`adopt-foreground-leftovers`](design-adopt-foreground-leftovers.md)<br>[Sketch](decision-adopt-foreground-leftovers.md) | tapestry · solid | T9 | 36m<br>35m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T11 | `draft` **The prime is the root beam: parent_beam names it, never empty** | [`prime-is-root-beam`](design-prime-is-root-beam.md) | darn · solid | T10 | 24m<br>9m | board, dobby, orch |
| T12 | `draft` **A stop button in the prime orch session that takes the whole hub down, behind a yes/no** | [`hub-stop-button`](design-hub-stop-button.md) | darn · solid | T11 | 4h<br>8m | reed, orch, tender |
| T13 | `draft` **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | darn · solid | T12 | 7d<br>2m | orch, reed, termwindow, shortcut |
| T14 | `draft` **Revise the remaining adopt-build members against what landed and what the operator decided since the split** | [`adopt-build-revision`](design-adopt-build-revision.md) | darn · docs | T3<br>T6 | 0m<br>0m | docs, batten, dobby, orch, fabric, tender |

#### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Own entries

The entries directly under this project and under none of its sub-projects.

#### Tasks (0)

Concrete and claimable; only a task can run.

##### Ready (0)

Waits on nothing open; can start now.

_None._

#### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (2)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 11d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 9d<br>6d | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 6d<br>4d | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (4)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 5d · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Study how to build and keep per-test coverage maps cheaply enough to pay off**<br>• `darn: writer fable[high]` | [`testmap-smart-build`](design-testmap-smart-build.md) | darn · docs | Darn-Write | neve | 16m<br>16m | testkit, gate, dobby |
| T4 | **Measure what sends coverage-map upkeep into whole-package rebuilds, and what a name-following selector would save** | [`testmap-miss-causes`](design-testmap-miss-causes.md) | darn · docs | Darn-Write | neve | 35m<br>35m | testkit, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 5d<br>4h | testkit, gate |
| E5 |  | **Coverage maps and test-suite trimming run only on command, never per landing** | [`testmap-on-command`](design-testmap-on-command.md) | 33m<br>22m | testkit, gate, dobby |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (13)</strong></summary>

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **A beam window opens a useless README, and markdown preview does not render** | [`ide-beam-readme-preview`](design-ide-beam-readme-preview.md) | darn · light | Finalize | neve | 29m<br>11m | ide |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P3 | 13d<br>35m | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 2d<br>2d | slack, creel, orch |

### Notes (10)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 6d<br>6d | gate, fabric |

#### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 7d<br>7d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 7d<br>3d | shed, loom, darn |

#### Features (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2h<br>2h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 4d<br>4d | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>9d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>11d | board, ide, orch |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 5w<br>9d | webster |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>9d | burler, shed, orch, quarry, report |

</details>
