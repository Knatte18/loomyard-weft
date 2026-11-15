# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-15 15:45 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 5d · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

## Order (operator, 2026-11-12)

1. Landed: the way to make and keep the maps, the upkeep pass (`lym gate testmap --upkeep`) and the prime dobby that starts it (`36621f0e`).
2. Landed: the pilot on one package and the verb that lists redundant-test candidates (`lym gate redundant`, `79f0eb4a`).
   Its report is `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`.
3. Open: the cut stage, `redundancy-sweep-existing`.
   The operator reads the candidate lists first; the pilot found that equal coverage often hides a different assertion, so the cut stays a reading filter, not a verdict.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Measure whether coverage maps pay off: incremental upkeep time, load on other runs, gate time saved** | [`testmap-payoff-measure`](design-testmap-payoff-measure.md) | darn · docs | Code-Bouncer | neve | 3h<br>3h | gate, testkit |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Measure what coverage-map upkeep costs, check how its state is shown, and write the operator guide**<br>• What the upkeep passes have cost, whether one call shows the maps state, and a short guide to using and keeping the maps | [`testmap-upkeep-cost-and-guide`](design-testmap-upkeep-cost-and-guide.md) | darn · docs | T5 | 2d<br>3h | gate, testkit, dobby |
| T5 | `draft` **Keep the coverage maps in each pair's weft branch, committed and pushed by lym gate testmap** | [`testmap-maps-in-weft`](design-testmap-maps-in-weft.md)<br>[Sketch](decision-testmap-maps-in-weft.md) | tapestry · solid | T6 | 2d<br>3h | gate, testkit, dobby |

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by** | [`selector-non-body-miss`](design-selector-non-body-miss.md) | 4d<br>3h | testkit, gate |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 5d<br>3h | testkit, gate |
