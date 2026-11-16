# Board

Rendered 2026-11-16 10:16 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-adopt-revision.md">P1 Revise adopt and the functions that move a hub between machines</a> (1)</strong></summary>

[`adopt-revision`](design-adopt-revision.md) · Age 22h · Owner `neve`

### [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (1)

[`adopt-build`](design-adopt-build.md) · Age 18h · Owner `neve`

#### Tasks (1)

Concrete and claimable; only a task can run.

##### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Merge the remaining adopt-build members into one design and split it again on what landed** | [`adopt-build-resplit`](design-adopt-build-resplit.md)<br>[Sketch](decision-adopt-build-resplit.md) | arras · light | Raddle-Burler-Review | neve | 7d<br>1h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

##### Ready (0)

Waits on nothing open; can start now.

_None._

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

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 12d · Owner `neve`

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
| T15 | `draft` **Codex as darn writer, replayed on a landed task, on OSL-1046** | [`codex-writer-bench-osl`](design-codex-writer-bench-osl.md) | darn · docs |  | 16h<br>16h | codex, shuttle |

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **A Codex session gets exactly what a Claude session in the same role gets**<br>• Skills, standing passages, instruction files, guards and prompt wording reach a Codex session in a role the same as a Claude one | [`codex-same-as-claude`](design-codex-same-as-claude.md)<br>[Sketch](decision-codex-same-as-claude.md) | 15h<br>32m | codex, shuttle, stencil, heddle |

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 10d<br>17h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 6d<br>15h | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (1)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 6d · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 6d<br>26m | testkit, gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (14)</strong></summary>

Owner `neve`

### Tasks (5)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Claude Code Mode for writer sessions, set per role**<br>• Turn on the experimental Code Mode, which batches tool calls in one program, for heavy writer roles only, after a bench | [`claude-code-mode`](design-claude-code-mode.md) | darn · docs | Open-Window | neve | 13m<br>0m | claude, shuttle, config |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (4)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P3 | 13d<br>18h | claude, shuttle, config |
| T2 | `draft` **Move Webster's batch machinery and run-plan state into a package both implementers import** | [`implementer-shared-run-plan`](design-implementer-shared-run-plan.md)<br>[Sketch](decision-implementer-shared-run-plan.md) | darn · solid |  | 0m<br>0m | webster, weaver |
| T5 | `draft` **Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam** | [`weaver-implementer`](design-weaver-implementer.md)<br>[Sketch](decision-weaver-implementer.md) | tapestry · solid | T2 | 0m<br>0m | webster, weaver |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 3d<br>3d | slack, creel, orch |

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
| F2 |  | **lym retro: one command that audits recent runs from their transcripts and proposes fixes**<br>• Turn the hand-set transcript audits into one command with a fixed lym-scoped question set, routing each finding to a check, a rubric line or a deletion | [`lym-retro`](design-lym-retro.md) | 18m<br>16m | friction, report, tools |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 20h<br>17h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 5d<br>31m | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>10d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>26m | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>10d | burler, shed, orch, quarry, report |

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T4 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md)<br>[Sketch](decision-weaver.md) | 5w<br>37m | webster, weaver |

</details>
