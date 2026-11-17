# Fix lym gate redundant kill counting and the map defects the trial found

`redundant-timeout-not-kill` · Task · bug, gate, testkit · starting · Worktree-Create

## Seen (orch, 2026-11-17)

The trial `map-tooling-trial` (`a666e8b3`, report `docs/research/speed/map-tooling-trial-2026-11-17/report.md`) ran `lym gate redundant` on `internal/boardengine`.
Of 30994 mutant test runs, 383 ended by timeout and 39 by the memory cap, and the verb counted every one as a kill.
A timeout or memory cap is not proof that the test caught the mutant, so these runs inflate the kills of the tests around a candidate,
which can make a candidate look redundant when it is not; the earlier survey found 27 of 30 sampled confirmations false.

## Idea

Count a run ended by timeout or the memory cap as unjudged, not as a kill, and report those mutants apart;
a candidate whose confirmation rests on such a run is listed as unconfirmed.

## Also: funcs order (was B4)

Found by the map-tooling-trial.

Two `lym gate test --wait --map internal/boardengine` runs on one commit (`b92c95ee`) wrote maps that hold the same sets per test but not the same bytes.
Beside `built_at` and `measured_at`, the `funcs` array of four tests lists the same ids in another order: `TestHandles_AssignedLowestFreeAndStable`, `TestHoldsRaddleFiling`, `TestRunLock`, `TestSetStatusReleasingProject_ReleasesDraftMembers`.
Set difference per test is empty in both directions, so only the order is unstable.
Two `internal/durfmt` runs differ in the timestamps alone.

A map that rewrites with a spurious diff each build makes any map committed or compared by bytes churn.
Sort each test's `funcs` and `blocks` before writing.

## Also: per-test seconds (was B5)

Found by the map-tooling-trial.

In the `internal/boardengine` map from `lym gate test --wait --map`, 132 of 136 tests record `seconds: 0` and the slowest records 0.29.
`lym gate redundant internal/boardengine` then lists every one of its 14 candidates with `seconds: 0`.
Its "greedy pass, slowest first" has no order to work from, and the standalone seconds it reports say nothing about what deleting a test saves.
Record each test's time at a resolution that separates tests, or say in the report that the figure is a floor.
