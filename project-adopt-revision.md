# P1 Revise adopt and the functions that move a hub between machines

Rendered 2026-11-15 16:24 UTC; every age below is as of then.
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

## [P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt](project-adopt-build.md) (6)

[`adopt-build`](design-adopt-build.md) · Age 26m · Owner `neve`

### Tasks (6)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder** | [`orch-resume-own-session`](design-orch-resume-own-session.md)<br>[Sketch](decision-orch-resume-own-session.md) | darn · solid | Drift-Verify | neve | 27m<br>26m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T6 | **One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked** | [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)<br>[Sketch](decision-run-fsm-tracked-records.md) | tapestry · solid | Plan-Write | neve | 27m<br>26m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay** | [`push-before-act-takeover`](design-push-before-act-takeover.md)<br>[Sketch](decision-push-before-act-takeover.md) | tapestry · solid | T6 | 27m<br>26m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence** | [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md)<br>[Sketch](decision-bolt-holds-catch-up.md) | tapestry · solid | T8<br>T3 | 27m<br>26m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

#### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs** | [`adopt-foreground-leftovers`](design-adopt-foreground-leftovers.md)<br>[Sketch](decision-adopt-foreground-leftovers.md) | tapestry · solid | T9 | 27m<br>26m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

#### Layer D (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **The prime is the root beam: parent_beam names it, never empty** | [`prime-is-root-beam`](design-prime-is-root-beam.md) | darn · solid | T10 | 15m<br>0m | board, dobby, orch |

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
