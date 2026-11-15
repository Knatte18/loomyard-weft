# Board

Rendered 2026-11-15 18:43 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-adopt-revision.md">P1 Revise adopt and the functions that move a hub between machines</a> (1)</strong></summary>

[`adopt-revision`](design-adopt-revision.md) · Age 7h · Owner `neve`

### [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (8)

[`adopt-build`](design-adopt-build.md) · Age 2h · Owner `neve`

#### Tasks (8)

Concrete and claimable; only a task can run.

##### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked** | [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)<br>[Sketch](decision-run-fsm-tracked-records.md) | tapestry · solid | Webster | neve | 2h<br>2h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Draft (7)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay** | [`push-before-act-takeover`](design-push-before-act-takeover.md)<br>[Sketch](decision-push-before-act-takeover.md) | tapestry · solid | T6<br>T14 | 2h<br>2h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T9 | `draft` **A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence** | [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md)<br>[Sketch](decision-bolt-holds-catch-up.md) | tapestry · solid | T8<br>T14 | 2h<br>2h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T10 | `draft` **lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs** | [`adopt-foreground-leftovers`](design-adopt-foreground-leftovers.md)<br>[Sketch](decision-adopt-foreground-leftovers.md) | tapestry · solid | T9<br>T14 | 2h<br>2h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T11 | `draft` **The prime is the root beam: parent_beam names it, never empty**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`prime-is-root-beam`](design-prime-is-root-beam.md) | darn · solid | T10<br>T14 | 2h<br>2h | board, dobby, orch |
| T12 | `draft` **A stop button in the prime orch session that takes the whole hub down, behind a yes/no**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hub-stop-button`](design-hub-stop-button.md) | darn · solid | T11<br>T14 | 7h<br>2h | reed, orch, tender |
| T13 | `draft` **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | darn · solid | T12<br>T14 | 7d<br>2h | orch, reed, termwindow, shortcut |
| T14 | `draft` **Revise the remaining adopt-build members against what landed and what the operator decided since the split** | [`adopt-build-revision`](design-adopt-build-revision.md) | darn · docs | T6 | 2h<br>2h | docs, batten, dobby, orch, fabric, tender |

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
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (4)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 11d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | `draft` **Codex as darn writer, replayed on a landed task, on OSL-1046** | [`codex-writer-bench-osl`](design-codex-writer-bench-osl.md) | darn · docs |  | 35m<br>34m | codex, shuttle |

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 9d<br>2h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 6d<br>4d | codex, shuttle |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A Codex session gets exactly what a Claude session in the same role gets**<br>• Skills, standing passages, instruction files, guards and prompt wording reach a Codex session in a role the same as a Claude one | [`codex-same-as-claude`](design-codex-same-as-claude.md) | 17m<br>17m | codex, shuttle, stencil, heddle |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (2)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 5d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Coverage maps and test-suite trimming run only on command, never per landing** | [`testmap-on-command`](design-testmap-on-command.md) | darn · solid | PR-Review | neve | 2h<br>1h | testkit, gate, dobby |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 5d<br>6h | testkit, gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (13)</strong></summary>

Owner `neve`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **The arbiter becomes a one-shot fixer that may edit, not only resume the halted row** | [`arbiter-one-shot-fixer`](design-arbiter-one-shot-fixer.md) | darn · solid | Code-Burler-Fix | neve | 58m<br>55m | arbiter, shed |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P3 | 13d<br>2h | claude, shuttle, config |
| T4 | `draft` **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | motif | T3 | 5w<br>45m | webster, weaver |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 2d<br>2d | slack, creel, orch |

### Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 6d<br>6d | gate, fabric |

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 8d<br>8d | heddle |

#### Features (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F2 |  | **Orch sessions turn on remote control by themselves, switchable per hub**<br>• Every orch session lym starts or resumes turns on Claude Code remote control, unless the hub switches it off | [`orch-auto-remote-control`](design-orch-auto-remote-control.md) | 23m<br>23m | orch, claude, config |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 4h<br>1h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 4d<br>4d | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>10d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>11d | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>10d | burler, shed, orch, quarry, report |

</details>
