# P1 Revise adopt and the functions that move a hub between machines

Rendered 2026-11-15 12:30 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-revision`](design-adopt-revision.md) · Age 47m · Owner `neve`

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

## Tasks (2)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Map every piece of state lym keeps, where it lives and whether git holds it** | [`lym-state-map`](design-lym-state-map.md) | darn · docs |  | 52m<br>47m | batten, tender, dobby, orch, fabric, loom |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Redesign adopt on one FSM of run state, pushed to GitHub at all times** | [`adopt-redesign`](design-adopt-redesign.md) | motif | T3 | 2d<br>46m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
