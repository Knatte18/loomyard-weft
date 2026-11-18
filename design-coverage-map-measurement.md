# Measure the coverage maps: build time, peak memory and whether their candidates hold

`coverage-map-measurement` · Task · research, gate, testkit · starting · Seed-IDE

## Why

Before the cut stage (`redundancy-sweep-existing`) the operator wants proof that the coverage maps are real and affordable.
The old `lym gate testmap` took about 45 minutes, and on 2026-11-17 a map build crashed the PC.
Since then `lym gate test --map` (`b92c95ee`) runs each test in its own process under `redundancy_test_memory_mb`,
and memory caps landed in `bd707e16`, `c75a2b5d` and `580b27a1`.
`9ce7233b` never reran `internal/boardengine`, so no one has confirmed its five known false candidates are gone.

## Runs (exact cap)

1. One full map build, `lym gate test --map --all`, run once.
   Record wall time, peak memory of the whole machine and of the largest test process, and whether any cap fired or any process was killed.
   Stop the build and report if free memory falls below 2 GB.
2. `lym gate redundant` on `internal/boardengine` and on one other package of the run's choosing, each run once and timed.
   Compare the candidates with the lists in `docs/research/speed/coverage-map-pilot-2026-11-12/report.md`,
   and check whether the five known false `boardengine` candidates are gone.
3. Hand crosscheck: for five candidates across the two packages, run the single test alone with coverage
   and compare the covered blocks with its map entry.

## Report

A report under `docs/research/speed/` with the three results, and a verdict:
whether the maps are fit for the cut stage, and whether `map-single-test-crosscheck` (E1) is needed before it
(needed if any crosscheck disagrees).
