# Coverage maps: build them, keep them fresh, and cut the test suite with them

`test-suite-trim` · Project · enhancement, testkit, gate, dobby

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

## Order (operator, 2026-11-12)

1. First the way to make and keep the maps: the upkeep pass (`lym gate testmap --upkeep`, landed) and the prime dobby that starts it (task `testmap-upkeep-dobby`, running; a running task cannot join a project).
2. Then the candidate list: `test-redundancy-candidates` builds the full set of maps and lists the redundant-test candidates per package, cutting nothing.
3. Only then the cut stage, `redundancy-sweep-existing`, from that list.
