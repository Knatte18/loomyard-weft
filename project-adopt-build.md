# P3 Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt

Rendered 2026-11-16 09:14 UTC; every age below is as of then.
Under: [P1 Revise adopt and the functions that move a hub between machines](project-adopt-revision.md).

[`adopt-build`](design-adopt-build.md) · Age 17h · Owner `neve`

beam

The build tasks raddle split from the Sketch `adopt-redesign`, run in a beam so they land on `main` together (operator, 2026-11-15).
No state between two members landing reaches `main`, so no installed lym runs half of the redesign.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Merge the remaining adopt-build members into one design and split it again on what landed** | [`adopt-build-resplit`](design-adopt-build-resplit.md) | arras · light | Discussion-Write | neve | 7d<br>10m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
