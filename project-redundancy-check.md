# P5 Redundant tests found mechanically: a coverage engine, a map per package, a gate report and its review

Rendered 2026-11-09 20:22 UTC; every age below is as of then.
Under: [Board](README.md).

[`redundancy-check`](design-redundancy-check.md) · Age 1d · Owner `neve`

Groups the entries filed from the Sketch `redundant-test-finder`: testcover-engine, testmap-verb, testmap-dobby-refresh, gate-redundancy-report, review-reads-redundancy.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T11 | **Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step** | [`review-reads-redundancy`](design-review-reads-redundancy.md)<br>[Sketch](decision-review-reads-redundancy.md) | darn · light | Darn-Write | neve | 1d<br>1d | testkit, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
