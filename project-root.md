# P0 Root: entries under no project

Rendered 2026-11-19 17:46 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Board viewer page with live editing through a loopback board server, opened by lym board open and a VS Code key** | [`board-viewer-server`](design-board-viewer-server.md)<br>[Sketch](decision-board-viewer-server.md) | tapestry · solid | Webster | neve | 1h<br>1h | board, ide, fabric, process, cwd |
| T4 | **Bug runs prove their test was red before the fix, and name the root cause in a trailer**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`bug-red-first-gate`](design-bug-red-first-gate.md) | darn · light | starting · Seed-Child | neve | 4h<br>3h | gate, testmap |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 10d<br>10d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A shell -c script that removes files stalls a run on a permission dialog**<br>• The shell guard does not look inside a shell -c script, so Claude Code asks for permission and the run stalls until someone answers | [`shellguard-sh-c-rm-prompt`](design-shellguard-sh-c-rm-prompt.md) | 1h<br>1h | process |
| B2 |  | **A hardenshed test leaves .lock in the package directory**<br>• A record write in a hardenshed test takes its lock beside a relative or empty path, so internal/hardenshed/.lock appears in the worktree and blocks Finalize | [`hardenshed-test-stray-lock`](design-hardenshed-test-stray-lock.md) | 4m<br>4m | gate |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 4d<br>1d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T10 | **Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired** | [`hardener-campaign`](design-hardener-campaign.md)<br>[Sketch](decision-hardener-campaign.md) | 8h<br>6h | shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs |
