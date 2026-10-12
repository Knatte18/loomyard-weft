# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING), fixed as proposed in `_lyx/plan/21-orch-pause.md`.
  `Watcher` keeps the time of its first tick in memory.
  The handled-with-no-Esc bullet now keys on an entry whose `GraceEnd` lies before that first tick.
  An entry the watcher was already running for, including one at grace 0, goes to the grace-end handling even when the first tick that reads it falls after `GraceEnd`.
  The at-`GraceEnd` bullet now reads "at or after `GraceEnd`" so that case reaches it.
  The test gains the row for a watcher created before an entry written at grace 0.
- **F2** (MEDIUM), fixed as proposed in `_lyx/plan/21-orch-pause.md`.
  `internal/orchengine#State` and `internal/orchengine#ResetForFreshLaunch` are on the Edit list.
  `State` gains `LaunchedAt` (`launched_at`), set by `ResetForFreshLaunch`.
  A new `pauseTick` bullet treats an entry whose `PausedAt` is not after `LaunchedAt` as the orch started under the entry: no interrupt, no record, no resume line.
  A zero `LaunchedAt` reads as launched before every entry.
  The Bound line names the rule, and the test gains the row for a `LaunchedAt` between `PausedAt` and `GraceEnd`.
- **F3** (LOW), fixed as proposed in `_lyx/plan/23-pause-cli.md`.
  The scenario is S12, with an S12 line after S11 in the Session log format block.
- **F4** (LOW), fixed as proposed in `_lyx/plan/17-batten-pause-reader.md`.
  Create gains `plan:internal/battencli#TestWire_PointsTheNetworkGateAtItsSlug`, and Edit gains `cmd/lyx/testdata/test-budget.yaml`.
  The Tests list describes the test, and the battencli budget row grows by one.

## Disputed

None.

## Deferred

None.
