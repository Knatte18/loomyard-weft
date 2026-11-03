```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **`describePushLockHolder` clause for no record, a live holder, a dead holder and a corrupt record.** `TestDescribePushLockHolder` rows NoRecord, LiveHolder, DeadHolder and Corrupt (`pushlockholder_test.go:32-35`). The DeadHolder row also pins that an unknown key is tolerated. An empty record file falls to the same `json.Unmarshal` error branch as Corrupt, so no separate row is needed.
- **The coalescing holder loop writes the record during `step`, names this pid in the busy error, removes the record on release and frees the lock.** `TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld` (`pushlockholder_test.go:55-85`).
- **`pushLockBusyError` still wraps `ErrPushLockBusy` and appends the holder clause.** The same test checks both with `errors.Is` and `strings.Contains` (`:66`). `pushanchored_integration_test.go:523` pins the wrapping on the real bounded push path.
- **`pushRidingOutBusyLock` retry behaviour.** `TestPushRidingOutBusyLock` has five rows (`landingdeps_test.go:538-544`):
  - a push that lands at once;
  - a push that lands after one busy attempt;
  - a non-busy failure at once;
  - a non-busy failure after a busy attempt;
  - busy through every pause, which returns the last busy error and sleeps once per pause.
- **`runHeld` now takes a release function.** The existing `coalesceFlagged` tests cover it, and the new coalesce test covers the release path.
- **Holder record in `tryRecordsLocks`: written on a weft, none on Bolt, removed on release.** None.
- **Holder record in `lockWeftForReplay`.** None.
- **`holdPushLock` failure paths: a record that cannot be written or removed is logged and the lock is still held and released.** None.
- **`PushTask` is wired through `pushRidingOutBusyLock` with `landingPushRetryPauses` and `time.Sleep`.** None.

**Findings**

1. **LOW, design.** `internal/fabricengine/originreconcilemoves.go:94-97` and `internal/fabricengine/pull.go:500`: two of the four sites that call `holdPushLock` have no test.
   - Dropping `holdPushLock` there would fail no test. The same goes for adding it on the Bolt branch.
   - Only the coalescing holder loop is exercised.
   - The summary at `_lym/landing/summary.md` claims this coverage ("the weft replay's lock, and origin reconcile's try-lock of a weft").
   - Evidence: `pushlockholder_test.go` drives only `coalesceFlagged`.
2. **LOW, design.** `internal/fabricengine/pushlockholder.go:44-46,48-50`: the Warn-and-continue paths of `holdPushLock` are untested.
   - These are the paths for a write failure and for a remove failure other than not-exist. The code's own contract says the lock is still held and released.
   - A regression that returned early or skipped `held.Release()` on a write error would pass.
   - This is a trust-boundary failure path, so it needs a row. For example, make `lockDir` a path whose parent is a file, then assert that the release function still frees the lock.
3. **LOW, design.** `internal/loomcli/landingdeps.go:346-350`: the `PushTask` wiring has no test.
   - A regression that dropped the `pushRidingOutBusyLock` wrap, or passed nil pauses, would pass.
   - I found no existing test that pins the `PushTask` closure's behaviour.
   - Whether to cover it is the handler's call, since a closure inside `pullRequestDepsGuardedBy` may be exempt by the surface rule.

No redundancy, wrong-assertion or static-prose findings. The five `pushRidingOutBusyLock` rows each cover a distinct path.

**Tool calls:** none. The diff and the manifest already held everything my lens needed.
