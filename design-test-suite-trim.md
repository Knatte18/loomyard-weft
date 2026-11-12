# Coverage maps: build them, keep them fresh, and cut the test suite with them

`test-suite-trim` · Project · enhancement, testkit, gate, dobby

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

## Order (operator, 2026-11-12)

1. Landed: the way to make and keep the maps, the upkeep pass (`lym gate testmap --upkeep`) and the prime dobby that starts it (`36621f0e`).
2. Landed: the pilot on one package and the verb that lists redundant-test candidates (`lym gate redundant`, `79f0eb4a`).
   Its report is `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`.
3. Open: the cut stage, `redundancy-sweep-existing`.
   The operator reads the candidate lists first; the pilot found that equal coverage often hides a different assertion, so the cut stays a reading filter, not a verdict.
