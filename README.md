# Board

Rendered 2026-11-22 11:23 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 5d · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>19h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric</a> (1)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 3d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Hardener campaign on fabric, one-repository layout**<br>• Hardener run on the one-repository part of fabric only<br>• `hardener: packages ["internal/fabricengine","internal/fabriccli","internal/gitrepo"]`<br>• `loom: notify_parent_on ["Harden-Bouncer","Harden-Burler-Fix"]` | [`harden-fabricengine`](design-harden-fabricengine.md) | hardener | Harden-Burler-Review | neve | 16h<br>1h | fabric |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (13)</strong></summary>

Owner `neve`

### Tasks (5)

Concrete and claimable; only a task can run.

#### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Orch remote control is off by default**<br>• lym turns on remote control for every orch it launches; it must stay off until the operator turns it on.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-rc-off-by-default`](design-orch-rc-off-by-default.md) | darn · light | Darn-Write | neve | 1m<br>1m | orch, config |
| T3 | **Sampler session loads the language skills before it edits code**<br>• The sampler spec carries no language skills, so the sampler edits code without the skills of the repository's languages.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`sampler-loads-language-skills`](design-sampler-loads-language-skills.md) | darn · light | Code-Burler-Review | neve | 1h<br>14m | loom, shuttle |
| T5 | **Board README: beam marker, no empty sections, no Notes level**<br>• Declutter the rendered board README in a sampler session | [`readme-declutter`](design-readme-declutter.md) | sampler | Sampler-Session | neve | 1h<br>1h | board |
| T6 | **lym report convergence stays within bounded memory**<br>• convergence grew to 9.5 GB in two minutes over a four-week window and froze the machine.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`convergence-report-memory`](design-convergence-report-memory.md) | darn · light | Code-Burler-Fix | neve | 36m<br>33m | report |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **A stale records tracking ref blocks a cancelled slug from running again**<br>• OriginCodeTip reads a deleted records branch through an unpruned tracking ref; dobby refuses the start<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`stale-records-ref-blocks-rerun`](design-stale-records-ref-blocks-rerun.md) | darn · light | T2 | 1h<br>1h | fabric, batten, dobby |

### Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge**<br>• Parent orch settles all halts of a project or beam; operator reached only for a severe halt or a merge-ready beam PR | [`orch-full-autonomy-mode`](design-orch-full-autonomy-mode.md)<br>[Sketch](decision-orch-full-autonomy-mode.md) | 2h<br>13m | orch, drift, landing, dobby, batten, pawl, config |

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 12d<br>12d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

#### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Hardener rounds may run fabric verbs in their own sandbox hubs**<br>• The task-session fabric guard refuses hardener rounds the verbs they must drive on sandbox hubs | [`harden-sandbox-fabric-verbs`](design-harden-sandbox-fabric-verbs.md) | 1h<br>1h | fabric, shed |
| B2 |  | **The board README renderer carries tests for pure cosmetics**<br>• A renderer of board markdown carries no layout tests; delete the README goldens and keep only tests of behaviour other code relies on. | [`board-readme-golden-tests`](design-board-readme-golden-tests.md) | 8m<br>4m | board |
| B3 |  | **Hardener live check passes packages go can resolve**<br>• hardenLiveArgs passes bare internal/x paths, which go reads as std packages, so harden-live fails at setup every round. | [`harden-live-bare-package-paths`](design-harden-live-bare-package-paths.md) | 58m<br>58m | gate, loom |

#### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 6d<br>4d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
