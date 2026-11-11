# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-11 15:56 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **testmap records each test's coverage across every in-module package its binary links** | [`testmap-cross-package`](design-testmap-cross-package.md)<br>[Sketch](decision-testmap-cross-package.md) | darn · solid | Code-Burler-Fix | neve | 6h<br>6h | gate, testkit, darn |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T14 | **Select an importing package's tests from its coverage map, or name the miss that runs it whole** | [`coverage-test-select`](design-coverage-test-select.md)<br>[Sketch](decision-coverage-test-select.md) | darn · solid | T4 | 6h<br>27m | gate, testkit, darn |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **A coverage gate scope runs changed packages whole and importing packages' selected tests** | [`gate-scope-coverage`](design-gate-scope-coverage.md)<br>[Sketch](decision-gate-scope-coverage.md) | darn · solid | T14 | 6h<br>27m | gate, testkit, darn |

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S2 |  | **Decide how the coverage maps are kept fresh, from the upkeep research** | [`testmap-upkeep-design`](design-testmap-upkeep-design.md)<br>[Sketch](decision-testmap-upkeep-design.md) | 1h<br>52m | testkit, gate, dobby |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 1d<br>29m | testkit, gate |
| E12 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 1d<br>29m | dobby, gate, testkit |
