# P1 Adopt as one resumable algorithm that never starts work on its own

Rendered 2026-11-11 06:33 UTC; every age below is as of then.
Under: [Board](README.md).

[`adopt-takeover`](design-adopt-takeover.md) · Age 1d · Owner `neve`

Groups the entries filed from the Sketch `adopt-robust`: adopt-worker-record, adopt-owner-handover, adopt-here-archives.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Adopt runs as a detached, step-recorded worker that pauses every adopted run and writes the owner before the deploy** | [`adopt-worker-record`](design-adopt-worker-record.md)<br>[Sketch](decision-adopt-worker-record.md) | tapestry · solid | Webster | neve | 1d<br>1d | batten, tender, dobby, orch, fabric, shortcut |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **The adopting machine's orch and dobby stay up and idle, and the old owner pauses, archives its unpushed tips and exits** | [`adopt-owner-handover`](design-adopt-owner-handover.md)<br>[Sketch](decision-adopt-owner-handover.md) | darn · light | T2 | 1d<br>1d | batten, tender, dobby, orch, fabric, shortcut |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **lym adopt --here behind the lym s shortcut, and lym adopt --archives lists the old owner's archives** | [`adopt-here-archives`](design-adopt-here-archives.md)<br>[Sketch](decision-adopt-here-archives.md) | darn · light | T4 | 1d<br>1d | batten, tender, dobby, orch, fabric, shortcut |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
