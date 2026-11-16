# P1 Revise adopt and the functions that move a hub between machines

Rendered 2026-11-16 11:10 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-revision`](design-adopt-revision.md) · Age 23h · Owner `neve`

## Goal

Revise the adopt module and every function that moves a hub and its runs between machines, so adopt works every time.
The operator's requirement and decisions stand in the member `adopt-redesign`:
every run's state is one FSM pushed to GitHub at all times, adopt runs in the foreground,
and nothing else happens in the hub until adopt is done and every failing test it found is fixed.

## Order

First the read-only study `lym-state-map` maps the state lym keeps.
Then the Motif `adopt-redesign` settles the design, and raddle splits its Sketch into the build tasks of this project.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (1)

[`adopt-build`](design-adopt-build.md) · Age 19h · Owner `neve`

### [P4 Merge the remaining adopt-build members into one design and split it again on what landed](project-adopt-build-resplit.md) (7)

[`adopt-build-resplit`](design-adopt-build-resplit.md) · Age 7d · Owner `neve`

#### Tasks (7)

Concrete and claimable; only a task can run.

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Draft (7)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **A transition pushes its records before the run acts, and a takeover entry in the run's records hands the run over** | [`transition-push-takeover`](design-transition-push-takeover.md)<br>[Sketch](decision-transition-push-takeover.md) | tapestry · solid |  | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T4 | `draft` **A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence** | [`machine-registry-holds-catchup`](design-machine-registry-holds-catchup.md)<br>[Sketch](decision-machine-registry-holds-catchup.md) | tapestry · solid | T3 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T8 | `draft` **lym adopt runs in the foreground over pushed state: hold, owner, pull, takeover, check, with --take-hold** | [`adopt-foreground`](design-adopt-foreground.md)<br>[Sketch](decision-adopt-foreground.md) | tapestry · solid | T4 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T9 | `draft` **The dobby tears down or reports leftover pairs** | [`leftover-pairs-teardown`](design-leftover-pairs-teardown.md)<br>[Sketch](decision-leftover-pairs-teardown.md) | darn · solid | T8 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T10 | `draft` **lym beam stop --all takes the hub down on this machine, behind a stop button**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hub-stop-all`](design-hub-stop-all.md)<br>[Sketch](decision-hub-stop-all.md) | darn · solid | T9 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T11 | `draft` **lym machine start brings back every hub this machine owns, from desktop launchers**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`machine-start-launchers`](design-machine-start-launchers.md)<br>[Sketch](decision-machine-start-launchers.md) | darn · solid | T9 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T12 | `draft` **The board prints the prime as the root beam `root`**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-prints-root-beam`](design-board-prints-root-beam.md)<br>[Sketch](decision-board-prints-root-beam.md) | darn · light | T8 | 47m<br>47m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

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

## Own entries

The entries directly under this project and under none of its sub-projects.

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
