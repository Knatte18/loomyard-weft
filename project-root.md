# P0 Root: entries under no project

Rendered 2026-11-22 15:30 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Per-task beam settings use one lookup from the worktree up**<br>• One lookup for per-task beam settings: the task worktree's own value, else its parent's up to the hub; pull_request and land_notice move onto it.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`per-task-settings-one-lookup`](design-per-task-settings-one-lookup.md) | darn · solid | Darn-Write | neve | 3h<br>3h | dobby, config, board, landing |
| T5 | **Board README: beam marker, no empty sections, no Notes level**<br>• Declutter the rendered board README in a sampler session | [`readme-declutter`](design-readme-declutter.md) | sampler | Drift-Verify | neve | 5h<br>5h | board |
| T8 | **Hardener rounds may run fabric verbs in their own sandbox hubs**<br>• The task-session fabric guard refuses hardener rounds the verbs they must drive on sandbox hubs<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`harden-sandbox-fabric-verbs`](design-harden-sandbox-fabric-verbs.md) | darn · light | Code-Burler-Fix | neve | 5h<br>4h | fabric, shed |
| T10 | **The board README renderer carries tests for pure cosmetics**<br>• A renderer of board markdown carries no layout tests; delete the README goldens and keep only tests of behaviour other code relies on.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-readme-golden-tests`](design-board-readme-golden-tests.md) | darn · light | Code-Bouncer | neve | 4h<br>4h | board |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **A stale records tracking ref blocks a cancelled slug from running again**<br>• OriginCodeTip reads a deleted records branch through an unpruned tracking ref; dobby refuses the start<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`stale-records-ref-blocks-rerun`](design-stale-records-ref-blocks-rerun.md) | darn · light | T2 | 5h<br>5h | fabric, batten, dobby |
| T11 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge** | [`orch-autonomy-mode`](design-orch-autonomy-mode.md)<br>[Sketch](decision-orch-autonomy-mode.md) | tapestry · solid | T3 | 3h<br>3h | orch, landing, loom, pawl, drift |

## Notes (4)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 7d<br>4d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T1 | **Orch remote control is off by default**<br>• lym turns on remote control for every orch it launches; it must stay off until the operator turns it on.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-rc-off-by-default`](design-orch-rc-off-by-default.md) | 4h<br>4h | orch, config |
