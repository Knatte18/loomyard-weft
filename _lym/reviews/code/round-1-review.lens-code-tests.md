```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**

1. `removeRunRecords` commits only while the records HEAD tracks the run.
   - Pinned by `TestRemoveRunRecords_CommitsTheRemovalOnlyWhileTheRecordsHeadTracksTheRun` (`cleanup_test.go`, added at about line 360). Its four rows are: seed tracked and on disk, status tracked and deleted, already gone and untracked, untracked and on disk.
   - Each row asserts `committed`, the commit count, run-directory removal and scratch emptied.
   - The "already gone and untracked" row is the reported bug, and it fails if the `tracked` guard is dropped.
   - The status-tracked row covers the first branch of `runRecordsTracked`, and the seed row covers the second.
2. `runRecordsTracked` extraction, reused by `RecordsPresent` in `cancelcmd.go`.
   - There is no new row, and none is needed because the behaviour is unchanged.
   - The new test exercises the helper through `removeRunRecords`.
   - The `buildCleanupFixture` change (`stubs.committed[SeedRel]`) keeps the existing `TestCleanup` rows green under the new guard. Done-run candidates now count as tracked, so their commits still happen.
3. `Admit` and `Network` under an admission.
   - Pinned by the "an admission passes every call of the process at once…" subtest in `hold_test.go`.
   - It asserts that `Admit` waits once on a live entry, that a later `Network` call under the admission does not sleep, and that two in-flight markers exist.
   - It also asserts that no markers remain after release, and that the next call is held again (`sleeps == 2`).
4. `HoldRemoteRefs` passes the gate before the lock and stays admitted until release.
   - Pinned by `TestHoldRemoteRefs_PassesThePauseGateBeforeTheLockAndStaysAdmittedUntilTheRelease` (`remoterefslock_integration_test.go`).
   - The lock is free while the gate waits, and a call under the lock passes with no new wait.
   - After `release()` the next call waits again.
5. `cancel --remove` after a plain cancel succeeds and the cancel marker is removed.
   - Only the `removeRunRecords` unit is pinned (item 1).
   - Nothing runs `RemoveRecords` or `cancelRun` over an already-removed records directory.
   - The `os.MkdirAll` of the status lock's directory is not exercised by any test with an empty recreated directory.
   - The goal's cause section names that empty directory, but `RemoveAll` makes the unit row equivalent to it.

**Findings**

- **LOW, design**, `internal/battencli/cleanup.go:292-295`, uncovered failure path.
  - `removeRunRecords` returns a plain `fmt.Errorf` when `deps.Committed` fails. It is neither a `removalCommitError` nor a `scratchEmptyError`, so `cleanupSlug` takes the `err != nil` case and skips the slug.
  - The stub's `Committed` never errors, so no row covers this new error path.
  - Evidence: `cleanup_test.go:55-60`, where `Committed` always returns a nil error. This is a gap in the new test, not in the production code.
- **LOW, design**, `internal/gitrepo/remoterefslock.go:32-34`, uncovered failure path.
  - The `leaveGate()` call when `lock.AcquireWriteLock` fails has no test. Without it the admission would leak and the process's network calls would pass the gate for the rest of its life.
  - Evidence: the integration test only covers the success path (`remoterefslock_integration_test.go`, new test).
- **NIT, design**, `internal/pauserecord/hold.go:424-431`, small gap.
  - `Admit` with no gate installed is untested. It still increments `admissions`, and a gate installed afterwards would then pass at once.
  - The doc comment states the no-gate behaviour ("waits for nothing and marks nothing"), but no row asserts it.
- **NIT, consistency**, partial overlap.
  - The "a call under the lock passes with no new wait" assertion in the `gitrepo` integration test repeats the `hold_test.go` admission subtest, which also asserts that a call under an admission does not sleep.
  - The `gitrepo` test is justified by what the `hold_test.go` subtest cannot reach: `HoldRemoteRefs` taking the gate before the lock, and release dropping the admission. I do not recommend deleting it.
  - The same test does not pin the release order (lock release, then gate leave), so a swapped order would still pass.

Test-economy note: the new `removeRunRecords` test is a table in a new top-level function, not a row in `TestCleanup`. It covers a different function (`removeRunRecords` called directly), so it is not a duplicate.

Tool calls: none. The manifest already held the diff, the test code and the production code.
