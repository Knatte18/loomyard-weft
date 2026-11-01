# P2 Fewer agent turns, Webster forks first

Rendered 2026-11-01 16:52 UTC; every age below is as of then.
Under: [Board](README.md).

[`agent-turns`](design-agent-turns.md) · Age 2h · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T14 | **Trial: agents settle every edit before the first, then gate once**<br>• An imperative heddle instruction: read, decide all edits, make them, gate once and fix the whole report in one round | [`plan-edits-before-gate`](design-plan-edits-before-gate.md) | darn | Finalize | neve | 2h<br>28m | heddle, stencil, claude |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
