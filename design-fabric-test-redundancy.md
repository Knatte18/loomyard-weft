# Which internal/fabricengine tier 1 tests protect nothing other tests do not: a coverage study

`fabric-test-redundancy` · Task · research, fabric, testkit · running · Seed-Child

## Seen (orch, 2026-11-08)

The fabric test speed family (project `fabric-test-speed-work`) cut `internal/fabricengine` tier 1 from 57 s to about 48 s inside a whole pass, against an expected 36 s (`docs/research/speed/fabric-test-speed-remeasure-2026-11-07.md`).
The package has about 1500 tier 1 leaf tests at a 60 ms median with no hotspot, 132 s run serially (`docs/research/speed/fabricengine-per-test-times-2026-11-07.md`).
Most of that time is git processes and hub builds the tests start.
The family folded only the overlaps an earlier report had read and confirmed; no one has asked of every test what it protects that no other test does.

## Idea (operator, 2026-11-08: "we do not need that many tests")

A measurement, run as darn with profile `docs`, on `internal/fabricengine` tier 1 only:

1. Run each top-level tier 1 test once on its own with `-coverprofile` scoped to `internal/fabricengine` (one coverage run per top-level test, no repeats).
2. List the tests whose covered blocks are all covered by other tests: candidates to drop or fold.
   Rank them by their time from the per-test times report.
3. List the tests that start git or build a hub but whose asserted behaviour is pure logic a tier 0 test could check without git.
4. For the top candidates of each list, read the test and say what behaviour it asserts that block coverage does not show, so a later task can decide.

The report lands under `docs/research/speed/` with the scripts beside it.
It deletes and moves no test: line coverage is not behaviour, and `fabricengine` manages worktrees and hubs, where a lost test can cost the operator data.
A later task decides the cuts from the report.
