# Coverage maps: build them, keep them fresh, and cut the test suite with them

`test-suite-trim` · Project · enhancement, testkit, gate, dobby

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
