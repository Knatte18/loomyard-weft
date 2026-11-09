# P1 Adopt as one resumable algorithm that never starts work on its own

Rendered 2026-11-09 19:48 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-takeover`](design-adopt-takeover.md) · Age 2m · Owner `neve`

Groups the entries filed from the Sketch `adopt-robust`: adopt-worker-record, adopt-owner-handover, adopt-here-archives.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Adopt runs as a detached, step-recorded worker that pauses every adopted run and writes the owner before the deploy** | [`adopt-worker-record`](design-adopt-worker-record.md)<br>[Sketch](decision-adopt-worker-record.md) | tapestry · solid |  | 2m<br>2m | batten, tender, dobby, orch, fabric, shortcut |

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **The adopting machine's orch and dobby stay up and idle, and the old owner pauses, archives its unpushed tips and exits** | [`adopt-owner-handover`](design-adopt-owner-handover.md)<br>[Sketch](decision-adopt-owner-handover.md) | darn · light | T2 | 2m<br>2m | batten, tender, dobby, orch, fabric, shortcut |
| T6 | `draft` **lym adopt --here behind the lym s shortcut, and lym adopt --archives lists the old owner's archives** | [`adopt-here-archives`](design-adopt-here-archives.md)<br>[Sketch](decision-adopt-here-archives.md) | darn · light | T4 | 2m<br>2m | batten, tender, dobby, orch, fabric, shortcut |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
