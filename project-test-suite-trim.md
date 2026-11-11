# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-11 11:56 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `osl-1046`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Research how the coverage maps are kept fresh: three independent brainstorms**<br>• Three agents, two Fable 5.2 and one Opus 5.5, each propose a smarter upkeep of the per-test coverage maps; one report lands<br>• `darn: writer fable[high]` | [`testmap-upkeep`](design-testmap-upkeep.md) | darn · docs |  | 1d<br>1h | testkit, gate, dobby |

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **testmap records each test's coverage across every in-module package its binary links** | [`testmap-cross-package`](design-testmap-cross-package.md)<br>[Sketch](decision-testmap-cross-package.md) | darn · solid | T3 | 2h<br>2h | gate, testkit, darn |
| T14 | `draft` **Select an importing package's tests from its coverage map, or name the miss that runs it whole** | [`coverage-test-select`](design-coverage-test-select.md)<br>[Sketch](decision-coverage-test-select.md) | darn · solid | T4 | 2h<br>2h | gate, testkit, darn |
| T15 | `draft` **A coverage gate scope runs changed packages whole and importing packages' selected tests** | [`gate-scope-coverage`](design-gate-scope-coverage.md)<br>[Sketch](decision-gate-scope-coverage.md) | darn · solid | T14 | 2h<br>2h | gate, testkit, darn |

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 1d<br>1d | testkit, gate |
| E12 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 1d<br>4h | dobby, gate, testkit |
