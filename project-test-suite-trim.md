# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-17 10:58 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 7d · Owner `neve`

Builds the first full set of coverage maps and uses them to cut the test suite:
cut existing redundant tests.
The maps are built only on command now; the gate no longer selects tests from them.

## Order (operator, 2026-11-12)

1. Landed: the way to make the maps (`lym gate testmap`).
   The upkeep pass (`--upkeep`) and the prime dobby that started it were retired by `testmap-on-command` (`72b26d1a`, 2026-11-15):
   the payoff report found the pass took 7.7 to 31 minutes after a landing and saved the gate 0 to 4.2 s.
   The maps are built only when the operator runs `lym gate testmap`, and the gate runs whole packages, with no `coverage` scope.
2. Landed: the pilot on one package and the verb that lists redundant-test candidates (`lym gate redundant`, `79f0eb4a`).
   Its report is `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`.
   The maps stay as the input of this verb.
3. Open: the cut stage, `redundancy-sweep-existing`.
   The operator reads the candidate lists first; the pilot found that equal coverage often hides a different assertion, so the cut stays a reading filter, not a verdict.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Build the coverage maps as a mode of the gate run, one process per test in parallel** | [`gate-builds-testmap`](design-gate-builds-testmap.md) | motif | Discussion-Bouncer | neve | 27m<br>20m | gate, testkit |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 7d<br>26m | testkit, gate |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
