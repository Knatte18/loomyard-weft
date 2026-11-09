# Run six of the longest serial internal/fabricengine tests in parallel

Tier 1 of `internal/fabricengine` spends most of its wall time in the serial phase, where tests without `t.Parallel` run one by one.
Most of the eight longest serial tests were serial only because they set a package-level variable the production code reads.
This change moves that state into a parameter the test passes, so six of the eight now call `t.Parallel`.
No production path changes what it does; production passes the value it used before.

## What moved where

- **Origin reconcile hooks.** `originCallHook` and `originVerdictHook` are gone.
  `ReconcileOptions` gains `CallHook` and `VerdictHook`, nil in production,
  and the ls-remote, fetch and verdict steps read them from the options they already take.
  `TestReconcileOrigin_Judge`, `TestReconcileOrigin_Ended` and `TestReconcileOrigin_Moves` pass the hooks in their options and run in parallel.
  A test helper, `moveReconcileWith`, passes options to the moving reconcile.
- **Anchor move seams.** `anchorFailHook` and `weftHeadSHA` are gone.
  An `anchorSeams` value, zero in production, sits on `Topology`,
  and `BeginAnchorChange` hands it to the `AnchorChange`, which passes it to the weft move.
  `SetAnchorFailHookForTest` and `SetWeftHeadSHAForTest` now set it on one hub's `Topology`.
  `TestAnchorChange` and every subtest of it run in parallel.
- **Removal gc threshold.** `gcAutoThreshold` is now a `Topology` field, which `NewTopology` sets to git's default of 6700.
  `SetGCAutoThresholdForTest` sets it on one `Topology`.
- **Failed-push notifier.** `SyncOptions` gains `Notifier`.
  A push's failed-push record mails through it, or through the process's installed notifier when it is nil,
  which is what every production caller passes.
  `TestPushFailureClasses` passes its fake through each push's options and runs in parallel.
  `TestFailedPushRecord` passes its fake through `RecordPushFailureNotifyingForTest` and runs in parallel too, since it set the same notifier.

## The two that stay serial

- **`TestWorktreeSeam_Events`.** It installs the process-wide worktree seam and captures the process-wide logger.
  The seam is installed once per process in production, and the clone, Add, Pull, the parent-pair fast-forward and the origin reconcile all fire it.
  Making it a parameter means threading it through `CloneOptions`, `SyncOptions`, `Topology` and `OriginSeams`, and through `fabriccli`'s clone, which is a larger change.
- **`TestReconcile_MigratesMarkerlessWefts`.** It swaps `spawnDetachedPushFn`.
  `Fabric.Commit`, `SpawnStatusPush`, the failed-remove respawn and the pending-flag respawn all read that seam,
  and four other test files set it.
  Moving it with all of them changes the signature of `SpawnStatusPush` and `SpawnPairPush` and their callers in `fabriccli`, `loomcli` and `tendercli`, which is a larger change.

## Design decisions

- `TestStoreHousekeeping` is split in two.
  Its rows that read no log run in the parallel `TestStoreHousekeeping`.
  The rows that assert on a log line move unchanged to `TestStoreHousekeeping_Warnings`, which stays serial because it captures the process-wide logger.
  These are the probe that sees other pairs' sessions, the failing probe and the failing gc.
  Every assertion of the old test is kept.
- The installed push notifier stays, since production installs it once per process and the detached push and teardown children run with no options.
  The remote-teardown notice tests still install it and stay serial.
  The Go runner finishes every serial top-level test, cleanup included, before the first parallel one starts,
  so no parallel test sees a value they install.

## Measuring

This change does not bench.
A later run of `docs/research/speed/fabricengine-shards-2026-11-09/serial-share.sh` on a one-process run's events gives the new serial phase against the 143 s the report measured on OSL-1046.
