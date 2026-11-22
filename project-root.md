# P0 Root: entries under no project

Rendered 2026-11-22 18:07 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **Orch procedure to move a running task by a move note and a fresh run**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-move-running-task`](design-orch-move-running-task.md)<br>[Sketch](decision-orch-move-running-task.md) | darn · docs | Darn-Write | neve | 1h<br>1h | orch, heddle |
| T6 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md) | motif | Discussion-Burler-Fix | neve | 1h<br>38m | batten, fabric |
| T11 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge** | [`orch-autonomy-mode`](design-orch-autonomy-mode.md)<br>[Sketch](decision-orch-autonomy-mode.md) | tapestry · solid | Plan-Write | neve | 6h<br>6h | orch, landing, loom, pawl, drift |

## Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Code review asks the parent for held decisions and BLOCKING findings at a spent budget**<br>• A held decision finding gates until the parent rules, and a spent budget with BLOCKING findings open escalates to the parent instead of auto-accepting<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`code-bouncer-asks-parent`](design-code-bouncer-asks-parent.md) | darn · light | T11 | 5m<br>1m | burler, loom, darn |
| T2 | **Per-task beam settings: drop the member-line exception and fix the open findings**<br>• land_notice off means no line for beam members too, and the six review findings per-task-settings-one-lookup landed open are fixed<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`per-task-settings-fixups`](design-per-task-settings-fixups.md) | darn · light | T11 | 4m<br>1m | dobby, config, reconcile, board |

## Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 15m<br>15m | burler, fabric |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A beam pull request updates its description after each member landing**<br>• Dobby runs the beam describe step itself when a member lands under a recorded pull request, instead of asking the orch through the landed line | [`beam-pr-describes-itself`](design-beam-pr-describes-itself.md) | 4m<br>1m | dobby, landing |

## Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>53m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
