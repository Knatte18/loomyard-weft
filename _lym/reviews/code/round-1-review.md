---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/failedpush.go:102
    summary: SyncOptions.notifier precedence and its fallback to the installed notifier have no test row
    fix: In internal/fabricengine, add a row to an existing push test, for example TestPushFailureClasses, that installs a fake notifier with InstallPushNotifier, pushes with SyncOptions{} and asserts the fake received one mail. The test then runs serial, so put the row in a serial test, such as the one in remoteteardown_integration_test.go that already installs a notifier. Add a second row that sets SyncOptions.Notifier to a different fake and asserts only that fake is mailed.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/topology.go:35
    summary: No test asserts that NewTopology carries defaultGCAutoThreshold; a zero value would silently disable the removal gc
    fix: In export_test.go add an accessor, for example GCAutoThresholdForTest(t *Topology) int. Add one assertion to TestStoreHousekeeping, or to a tier 0 test, that NewTopology(cfg) returns a Topology whose threshold equals 6700, git's default gc.auto, without overriding it.
---

## Verdict

No finding is BLOCKING.
The diff makes six of the eight named tests parallel, and it names the cause for the two it leaves serial, as the decision record's task allows.

## Focus directive settled

- **Scope against the eight tests.**
  `TestPushFailureClasses`, `TestAnchorChange`, the three `TestReconcileOrigin_*` tests and `TestStoreHousekeeping` call `t.Parallel`.
  `TestWorktreeSeam_Events` and `TestReconcile_MigratesMarkerlessWefts` stay serial, each with its cause named in `_lym/landing/summary.md`.
  The record allows a test that stays serial when the cause cannot be removed without a larger change, so this is not a scope defect.
- **`failedpush.go` and the notifier.**
  `installedPushNotifier` is still read by `installedNotifier()` and `remoteteardown.go`.
  Every test that installs it is serial: `recordNotices`, `remoteteardown_integration_test.go`, and the `orchcli` tests, which sit in another package.
  A parallel test reaches the variable only when `SyncOptions.Notifier` is nil, and it is then nil because no serial test is running.
  Every test that set the notifier moved: `TestFailedPushRecord` and `TestPushFailureClasses`.
- **Production behavior.**
  `NewTopology` is the only `Topology` constructor, and it sets 6700.
  The hooks and `anchorSeams` are nil or zero in production.
  The `ReconcileOptions` hooks are nil in production.
  Only the source of each input moved.
- **`export_test.go` and `TestStoreHousekeeping`.**
  The log-reading rows moved unchanged into `TestStoreHousekeeping_Warnings`.
  The "empty answer packs" row keeps its pack assertion.
  The rows that moved still assert keys kept, stores unpacked and the log line, so no assertion is dropped or loosened.
- **Remaining serial tests.**
  The serial tests that touch `installedPushNotifier`, the logger, `spawnDetachedPushFn`, the worktree seam or the environment run to completion before the first parallel test starts, so a newly parallel test never sees their values.
  The remaining package variables the focus names, `originCallHook` and `gcAutoThreshold`, no longer exist.
- **Callers of changed signatures.**
  `addanchor.go`, `anchorchange.go`, `remove.go` and `pushclassified.go` are updated.
  `RecordPushFailure` keeps its signature, so the `fabriccli`, `orchcli` and `tendercli` callers are unchanged, and nothing outside `internal/fabricengine` is edited.

## Findings

### [MEDIUM] SyncOptions.notifier fallback is unpinned

**Location:** internal/fabricengine/failedpush.go:102
**Issue:** `SyncOptions.notifier` returns `Notifier`, or the installed notifier when `Notifier` is nil. Production passes nil everywhere, so the fallback is the production path.
Every `TestPushFailureClasses` row sets `Notifier` through `fake.options`, so a change that ignores the installed notifier, or that lets a nil `Notifier` shadow it, fails no test.

### [LOW] NewTopology's default gc threshold is unpinned

**Location:** internal/fabricengine/topology.go:35
**Issue:** Every housekeeping test overrides the threshold with `SetGCAutoThresholdForTest(..., 1)`, so a `NewTopology` that left the field at zero would pass them all.
A zero threshold passes `gc.auto=0` and disables the removal gc in production.

## Rejected

- The notifier is now read before the record lock in `RecordPushFailure`: a swap between the read and the lock has no observable effect, so it is no defect.
- Both ls-remote goroutines call the same `CallHook` concurrently: the old global hook ran under the same concurrency, and `callLog` tolerates it.
