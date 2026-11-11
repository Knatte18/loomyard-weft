# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-11 07:26 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T17 | **A verify gate scopes to nearly the whole suite; scope it by the coverage map**<br>• A diff in a widely imported package reruns tiers 0 to 2 across the suite, 30 to 40 minutes on OSL-1046. | [`gate-scope-by-coverage-map`](design-gate-scope-by-coverage-map.md) | motif | Discussion-Write | neve | 1d<br>1d | gate, testkit, darn |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 1d<br>1d | dobby, gate, testkit |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 1d<br>1d | testkit, gate |
| E5 |  | **Decide how the coverage maps are kept fresh: triggers, batch, staleness and sharing across machines** | [`testmap-upkeep`](design-testmap-upkeep.md) | 1d<br>1d | testkit, gate, dobby |
