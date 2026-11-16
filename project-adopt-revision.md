# P1 Revise adopt and the functions that move a hub between machines

Rendered 2026-11-16 09:04 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-revision`](design-adopt-revision.md) · Age 21h · Owner `neve`

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

[`adopt-build`](design-adopt-build.md) · Age 17h · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Merge the remaining adopt-build members into one design and split it again on what landed** | [`adopt-build-resplit`](design-adopt-build-resplit.md) | arras · light |  | 7d<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

### Notes (0)

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
