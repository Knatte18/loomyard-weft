# fabricengine: run six serial tests in parallel via injected seams

Six of the eight longest serial `internal/fabricengine` tests now run in parallel,
which shrinks the serial phase of the tier 1 run.
Each was serial because it set a package-level variable that production code reads.
That state is now a parameter the test passes, so parallel tests no longer see each other's values.
Production behavior is unchanged; only where the inputs come from moves.

## What changes

- Push notifier: `SyncOptions` gains a `Notifier` field.
  `recordPushFailure` takes the notifier as an argument, and `RecordPushFailure` passes the installed one.
  A nil `Notifier` falls back to the process's installed notifier.
  Made parallel: `TestPushFailureClasses`, `TestReconcileOrigin_Judge`.
- Origin call and verdict hooks: `originCallHook` and `originVerdictHook` are gone.
  `ReconcileOptions` carries `CallHook` and `VerdictHook`, and the reconcile paths read them from there.
  Made parallel: `TestReconcileOrigin_Ended`, `TestReconcileOrigin_Moves`, together with the other tests that set these hooks.
- Anchor move failures: `anchorFailHook` and `weftHeadSHA` are replaced by an `anchorSeams` value.
  It lives on `Topology`, is copied into each `AnchorChange` when it begins, and is passed to `moveWeftContent`.
  The zero value is production.
  Made parallel: `TestAnchorChange`.
- gc threshold: `gcAutoThreshold` becomes the constant `defaultGCAutoThreshold`.
  `Topology` holds the live value and passes it to `housekeepStores`.
  Made parallel: `TestStoreHousekeeping`.
- `export_test.go` setters now take the `*Topology` they configure, so tests inject per instance.

## Still serial

- `TestReconcile_MigratesMarkerlessWefts`: `SwapPushRecorderForTest` sets the package-level `spawnDetachedPushFn`.
  Moving it is a larger change than this task allows.
- `TestWorktreeSeam_Events`: it installs the process-wide worktree seam and captures the process logger.
  Neither can be moved into a parameter without a larger change.

## Design decisions

- Seams sit on `Topology` and the options types the code paths already take, rather than on new types threaded everywhere.
- The installed push notifier stays, because the detached push path still reads it.
  `TestSyncOptionsNotifier` is new and pins the precedence: `Notifier` first, the installed notifier when it is nil.
  It installs the process notifier, so it is itself serial.

## To check by hand

- The tier 1 gate on `./internal/fabricengine` with `-race` was not run for this description; run it,
  since a missed shared state shows there.
- Measure the new serial phase with `docs/research/speed/fabricengine-shards-2026-11-09/serial-share.sh`
  on a one-process run's events, against the 143 s baseline from OSL-1046.
  That figure comes from a machine where Cortex XDR slows every new binary, so other machines will show less.
