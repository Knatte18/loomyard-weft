# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-11 19:58 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **An upkeep pass runs under a slot it yields to a gate, side by side where tests declare it, and writes per round** | [`testmap-upkeep-runs`](design-testmap-upkeep-runs.md)<br>[Sketch](decision-testmap-upkeep-runs.md) | darn · solid | Code-Burler-Fix | neve | 2h<br>2h | testkit, gate, dobby |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **The prime dobby starts an upkeep pass on remaining work, and an anchor move ends it** | [`testmap-upkeep-dobby`](design-testmap-upkeep-dobby.md)<br>[Sketch](decision-testmap-upkeep-dobby.md) | darn · solid | T4 | 2h<br>2h | testkit, gate, dobby |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 2d<br>4h | testkit, gate |
