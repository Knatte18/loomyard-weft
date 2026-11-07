# Fold overlapping internal/fabricengine tests

Folds the overlapping `internal/fabricengine` tests that the tier 1 report read, to cut about 6 s of the package's serial time and about 2 s of wall.
Only test files change; no production code does.
Each fold keeps every assertion its folded row alone made, the "alone pins" column of the report's section 4.

## What changes

- `TestVerbCases_CleanState` is removed.
  Its three preconditions now run in `runCell` (`livestate_matrix_test.go`) through `assertArrangePreconditions`, right after the verb's arrange and before the state applies.
  The preconditions are that Add's launcher blocker is a directory, Checkout's branch differs from the checked-out one, and Pull's warp bare has advanced.
  Each check keys on a fixture field only the arranging verb sets, so the loop has no per-verb branch.
- `TestVerbCases_ArrangeSetsItsPreconditionField` replaces the removed test's role as a guard.
  The shared check skips an empty field, so an Arrange that stopped setting its field would leave the column vacuous without failing.
  This test pins that Add, Checkout and Pull each set theirs.
- `TestAnchorChange`: the three refusal rows that need no setup run on one shared `hubforge.CopyHub`.
  The refusal assertions move into `requireAnchorRefusal`, which the remaining setup rows share.
- `TestStoreHousekeeping`: the Remove-packs subtest and the gc-prune subtest become one row on the default nil probe.
  That row asserts the packed stores, the housekeeping keys, the prune of the two-day-old unreachable object and the keeping of the fresh one.
  The in-flight probe table gains the empty-answer row, which keeps the explicit empty-answer packing pin.
- `TestReconcileOrigin_Moves`: the `owned_elsewhere` row runs its replies and scopes on one pair.
- `TestPushFailureClasses`:
  - The rewound settle rows assert the hold-back on their first push through `requireRewoundHoldBack`, so the separate hold-back row goes.
  - The two backoff rows (detached prime weft and parked pair) become one table.
- `TestBolt_PullThenCommitWritten`: the behind-only/two-writes, seed-commit and conflicting-commit subtest pairs become two-row tables.

## Design decisions

- The rows folded are the ones the report read or found again by name; other survey rows stay as they were.
- The conflicting-commit table shares one `logcapture.Capture`, because the logger's sink is global and a capture per parallel row would replace the other's.
- The anchor-change shared-hub rows run in turn on one hub, not in parallel, since a refusal changes nothing and the hub is shared.

## To check by hand

- `internal/fabricengine/housekeeping_integration_test.go`: the nil-probe packing pin.
  The default-probe row and the empty-answer row now cover it together; confirm the folded Remove-packs subtest lost no assertion.
- Compare the folds against the report's "alone pins" column, row by row, as the acceptance criteria ask.
- No regression check ran for this task, and the timing gain has not been measured on this branch.
  Run the `internal/fabricengine` tests at tier 1 and compare against `main`.
