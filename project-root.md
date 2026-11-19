# P0 Root: entries under no project

Rendered 2026-11-19 13:12 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Survey Matt Pocock's skills repository for ideas that improve LoomYard**<br>• Read mattpocock/skills (main and the constrain-skill branch) and report what LoomYard could adopt<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pocock-skills-survey`](design-pocock-skills-survey.md) | darn · docs | Code-Bouncer | neve | 7m<br>7m |  |
| T3 | **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | motif | Discussion-Burler-Review | neve | 1d<br>48m | board, ide, fabric |
| T4 | **Two-repository hubs: status commits skip during weft replay, and the layout probe spams and costs git calls**<br>• Undo the f06acf39 merge-state probe widening for two-repository hubs, and make the layout probe quiet and cached<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`two-repo-merge-probe-regression`](design-two-repo-merge-probe-regression.md) | darn · light | landing · Finalize | neve | 57m<br>57m | fabric |
| T13 | **Settings to route code review and drift judge rulings to the run's parent**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`parent-review-code-and-drift`](design-parent-review-code-and-drift.md) | darn · solid | Code-Burler-Fix | neve | 3h<br>1h | loom, drift, burler |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired** | [`hardener-campaign`](design-hardener-campaign.md)<br>[Sketch](decision-hardener-campaign.md) | tapestry · solid | T13 | 3h<br>1h | shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs |

## Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 10d<br>10d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

### Features (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F2 |  | **Send a run agent questions for the operator to the run parent instead**<br>• A setting that routes every question a run agent would ask the operator in its pane to the parent by creel, with the answer delivered back | [`agent-questions-to-parent`](design-agent-questions-to-parent.md) | 3h<br>3h | loom, shuttle |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>1d | batten, loom, board |
| F4 |  | **Mail the run parent when the run reaches a named row**<br>• A subscription verb or run_settings key so lym mails the parent when a run reaches a named producer, in place of polling | [`parent-notify-on-row`](design-parent-notify-on-row.md) | 3h<br>3h | loom, shed |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
