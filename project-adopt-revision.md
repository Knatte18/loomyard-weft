# P1 Revise adopt and the functions that move a hub between machines

Rendered 2026-11-15 15:57 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-revision`](design-adopt-revision.md) · Age 4h · Owner `neve`

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

## [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (0)

[`adopt-build`](design-adopt-build.md) · Age 0m · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Own entries

The entries directly under this project and under none of its sub-projects.

### Tasks (5)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder** | [`orch-resume-own-session`](design-orch-resume-own-session.md)<br>[Sketch](decision-orch-resume-own-session.md) | darn · solid |  | 0m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T6 | `draft` **One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked** | [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)<br>[Sketch](decision-run-fsm-tracked-records.md) | tapestry · solid |  | 0m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T8 | `draft` **Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay** | [`push-before-act-takeover`](design-push-before-act-takeover.md)<br>[Sketch](decision-push-before-act-takeover.md) | tapestry · solid | T6 | 0m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T9 | `draft` **A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence** | [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md)<br>[Sketch](decision-bolt-holds-catch-up.md) | tapestry · solid | T8<br>T3 | 0m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T10 | `draft` **lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs** | [`adopt-foreground-leftovers`](design-adopt-foreground-leftovers.md)<br>[Sketch](decision-adopt-foreground-leftovers.md) | tapestry · solid | T9 | 0m<br>0m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T4 | **Redesign adopt on one FSM of run state, pushed to GitHub at all times** | [`adopt-redesign`](design-adopt-redesign.md)<br>[Sketch](decision-adopt-redesign.md) | 2d<br>1h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
