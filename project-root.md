# P0 Root: entries under no project

Rendered 2026-11-18 18:50 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (7)

Concrete and claimable; only a task can run.

### Running (6)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Find why VS Code keeps showing a stale board file, and fix it**<br>• Find the cause first, most likely in VS Code settings or how the workspace reaches the board files; change lym code only if the cause is there<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-view-not-refreshing`](design-board-view-not-refreshing.md) | darn · light | Darn-Write | neve | 35m<br>6m | board, ide |
| T4 | **Board entry imported from an issue loses the evidence the issue names, and an issue body can arrive empty**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`import-loses-issue-evidence`](design-import-loses-issue-evidence.md) | darn · light | Darn-Write | neve | 2h<br>3m | board, selfreport |
| T5 | **Bench the explorer subagents of the Discussion and Plan writers: with against without**<br>• Same frozen inputs, the writers run with and without the Agent-tool explore and lookup subagents; compare cost, time and plan or record quality<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`plan-writer-explorer-forks`](design-plan-writer-explorer-forks.md) | darn · docs | Darn-Write | neve | 6h<br>40m | loom, stencil |
| T6 | **Fabric single-repo mode: the weft lives inside the warp repo, no piggyback repo**<br>• Let a hub run with _lym and .lym as real directories in the warp repo instead of junctions to a separate weft repo, same as today from outside | [`fabric-single-repo-mode`](design-fabric-single-repo-mode.md) | motif | halted · Discussion-Bouncer | neve | 1h<br>35m | fabric |
| T7 | **A beam session loses the pawl run checklist when the beam overview takes the overview window**<br>• Bench arm of plan-writer-explorer-forks; never release | [`wxb-plan-with`](design-wxb-plan-with.md)<br>[Sketch](decision-wxb-plan-with.md) | tapestry | Plan-Write | neve | 32m<br>32m |  |
| T8 | **A beam session loses the pawl run checklist when the beam overview takes the overview window**<br>• Bench arm of plan-writer-explorer-forks; never release | [`wxb-plan-without`](design-wxb-plan-without.md)<br>[Sketch](decision-wxb-plan-without.md) | tapestry | Plan-Write | neve | 32m<br>32m |  |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>23h | board, ide, fabric |

## Notes (5)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 23h<br>23h | shuttle |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>10h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
