# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-16 09:15 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 6d · Owner `neve`

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

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | `draft` **Rebuild every coverage map, time it, and list the redundant-test candidates of each package**<br>• One timed full lym gate testmap build and lym gate redundant on every package, reported for the operator to read before any cut | [`redundancy-candidates-survey`](design-redundancy-candidates-survey.md) | darn · docs | T8 | 5m<br>0m | gate, testkit |
| T8 | `draft` **lym gate testmap builds in any worktree, with maps kept in that worktree**<br>• Build coverage maps at the HEAD of a task worktree and keep them in its .lym or .scratch, so a survey run is not cut by landings and throws the maps away after | [`testmap-any-worktree`](design-testmap-any-worktree.md) | darn · light |  | 0m<br>0m | gate, testkit |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 6d<br>14h | testkit, gate |
