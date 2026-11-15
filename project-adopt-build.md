# P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt

Rendered 2026-11-15 16:37 UTC; every age below is as of then.
Under: [P1 Revise adopt and the functions that move a hub between machines](project-adopt-revision.md).

[`adopt-build`](design-adopt-build.md) · Age 39m · Owner `neve`

beam

The build tasks raddle split from the Sketch `adopt-redesign`, run in a beam so they land on `main` together (operator, 2026-11-15).
No state between two members landing reaches `main`, so no installed lym runs half of the redesign.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (9)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder** | [`orch-resume-own-session`](design-orch-resume-own-session.md)<br>[Sketch](decision-orch-resume-own-session.md) | darn · solid | Code-Burler-Fix | neve | 39m<br>39m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T6 | **One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked** | [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)<br>[Sketch](decision-run-fsm-tracked-records.md) | tapestry · solid | Plan-Burler-Review | neve | 39m<br>39m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (7)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay** | [`push-before-act-takeover`](design-push-before-act-takeover.md)<br>[Sketch](decision-push-before-act-takeover.md) | tapestry · solid | T6<br>T14 | 39m<br>4m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T9 | `draft` **A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence** | [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md)<br>[Sketch](decision-bolt-holds-catch-up.md) | tapestry · solid | T8<br>T3<br>T14 | 39m<br>3m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T10 | `draft` **lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs** | [`adopt-foreground-leftovers`](design-adopt-foreground-leftovers.md)<br>[Sketch](decision-adopt-foreground-leftovers.md) | tapestry · solid | T9<br>T14 | 39m<br>3m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |
| T11 | `draft` **The prime is the root beam: parent_beam names it, never empty** | [`prime-is-root-beam`](design-prime-is-root-beam.md) | darn · solid | T10<br>T14 | 28m<br>3m | board, dobby, orch |
| T12 | `draft` **A stop button in the prime orch session that takes the whole hub down, behind a yes/no** | [`hub-stop-button`](design-hub-stop-button.md) | darn · solid | T11<br>T14 | 5h<br>3m | reed, orch, tender |
| T13 | `draft` **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | darn · solid | T12<br>T14 | 7d<br>3m | orch, reed, termwindow, shortcut |
| T14 | `draft` **Revise the remaining adopt-build members against what landed and what the operator decided since the split** | [`adopt-build-revision`](design-adopt-build-revision.md) | darn · docs | T3<br>T6 | 4m<br>4m | docs, batten, dobby, orch, fabric, tender |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
