# P0 Root: entries under no project

Rendered 2026-11-17 12:51 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T21 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | motif | Discussion-Burler-Review | neve | 6w<br>1h | board, ide, orch |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>1d | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 4d<br>4d | slack, creel, orch |

## Notes (5)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Record Codex edits for the edit-loop watch**<br>• Codex runs record no edits, so the edit-loop watch never notifies their parent | [`codex-edit-loop-recorder`](design-codex-edit-loop-recorder.md) | 31m<br>31m | codex, shuttle |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 1d<br>1d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>11d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
