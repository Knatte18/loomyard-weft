# P0 Root: entries under no project

Rendered 2026-11-22 17:20 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Each run transition waits for a push to origin, so a run starts about a minute slower**<br>• Since 71d6ee1a0 every batten row pushes its records before acting; a run start went from ~7 s to ~60 s.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`transition-push-slows-rows`](design-transition-push-slows-rows.md) | darn · light | Code-Burler-Fix | neve | 34m<br>23m | batten, fabric |
| T3 | **Per-task beam settings use one lookup from the worktree up**<br>• One lookup for per-task beam settings: the task worktree's own value, else its parent's up to the hub; pull_request and land_notice move onto it.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`per-task-settings-one-lookup`](design-per-task-settings-one-lookup.md) | darn · solid | Code-Burler-Review | neve | 5h<br>5h | dobby, config, board, landing |

## Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Orch procedure to move a running task by a move note and a fresh run**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-move-running-task`](design-orch-move-running-task.md)<br>[Sketch](decision-orch-move-running-task.md) | darn · docs | T3 | 28m<br>28m | orch, heddle |
| T6 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md) | motif | T2 | 23m<br>19m | batten, fabric |
| T11 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge** | [`orch-autonomy-mode`](design-orch-autonomy-mode.md)<br>[Sketch](decision-orch-autonomy-mode.md) | tapestry · solid | T3 | 5h<br>5h | orch, landing, loom, pawl, drift |

## Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>6m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
