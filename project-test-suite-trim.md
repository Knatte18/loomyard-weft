# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-18 09:31 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 8d · Owner `neve`

Builds the first full set of coverage maps and uses them to cut the test suite:
cut existing redundant tests.
The maps are built only on command now; the gate no longer selects tests from them.

## Order (operator, 2026-11-12)

1. Landed: the way to make the maps.
   The upkeep pass (`--upkeep`) and the prime dobby that started it were retired by `testmap-on-command` (`72b26d1a`, 2026-11-15):
   the payoff report found the pass took 7.7 to 31 minutes after a landing and saved the gate 0 to 4.2 s.
   `gate-test-map-mode` (`b92c95ee`, 2026-11-17) replaced `lym gate testmap` with `lym gate test --map`,
   which runs each test in its own process, in parallel, and writes each package's map whole.
   The maps are built only on command, and the gate runs whole packages, with no `coverage` scope.
2. Landed: the pilot on one package and the verb that lists redundant-test candidates (`lym gate redundant`, `79f0eb4a`).
   Its report is `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`.
   The maps stay as the input of this verb.
3. Open: the cut stage, `redundancy-sweep-existing`.
   The operator reads the candidate lists first; the pilot found that equal coverage often hides a different assertion, so the cut stays a reading filter, not a verdict.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 8d<br>14h | testkit, gate |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **No lym check compares a map entry with a single-test coverage run**<br>• The trial cross-checked by hand; blocks have no line range and funcs have no profile counterpart | [`map-single-test-crosscheck`](design-map-single-test-crosscheck.md) | 18h<br>1h | gate |
