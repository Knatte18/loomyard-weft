```yaml
lens: code-tests
outcome: reported
```

**Task: code-tests lens — for each behaviour the diff adds or changes, the test that pins it, or none.**

Per behaviour:
1. **The run-end stamp is set when a done run is cleaned up.** Pinned by the added `ReadNewestRunEnd` assertion in `internal/shuttleengine/wait_test.go:445`, in the done-cleanup case of `TestRun_Wait_Classification`. With the run directory removed and no record left, a missing stamp reads as the zero time and the assertion fails.
2. **The stamp is set on the other finalize outcomes (died, timeout, api-error, kept-pane done).** None. The assertion sits only in the cleaned-up done case, which is the only case where a run record cannot stand in for it. See finding 1.
3. **`ReadNewestRunEnd` returns the later of the stamp and the newest ended record.**
   - Pinned by `TestReadNewestRunEnd` in `internal/shuttleengine/activity_test.go`: `TheEndStampOfARemovedRunCountsWithNoRecordLeft`, `AnEndStampNewerThanEveryRecordWins` and `ARecordNewerThanTheEndStampWins`.
   - An absent root is pinned by the existing missing-root subtest.
4. **An unreadable stamp (any stat error other than absent) falls back to the records, and a failed `stampRunEnd` only warns.** None. Both paths are swallowed or logged, and no row injects a stat or write failure. See finding 3.
5. **`gateProcessLive` counts a registered `RoleGate`, `gate-fixer` or numbered `verify-fixer-2` process, and not another role.** Pinned by the added loop in `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` (`internal/battencli/wire_seams_test.go`). The rows are loop false, gate true, gate-fixer true and verify-fixer-2 true.
6. **`gateProcessLive` counts only processes registered under this run's pair slug.** None. See finding 2.
7. **A dead or recycled pid does not read as live.** No row in the diff. `runproc.List`'s own pruning is the behaviour that decides it, and the manifest does not show that package's tests. See finding 4.
8. **An error from `runproc.List` propagates out of `readAgentActivity`.** None, but it is a plain pass-through.
9. **`runnotice.go` change.** Comment only, so nothing to pin. The threshold behaviour the focus names is unchanged. `internal/dobbyengine/runnotice_test.go:230` already pins a running writer with `waitLive` true holding off the quiet notice.

Findings:

1. **LOW, design — no row for the stamp on non-done finalize paths.**
   - Location: `internal/shuttleengine/wait_test.go:445`; behaviour at `internal/shuttleengine/wait.go:1927`.
   - Summary: only the cleaned-up done case asserts the stamp. A regression that stamped only on done would pass every test, though died, timeout and api-error runs keep their record and are covered by its mtime. The gap matters only if the stamp is meant to be the end signal for every outcome, as the commit message and the `doc.go` addition claim ("every finalize").
   - Evidence: the assertion is placed before the `cleanupDiedCaptureKept` case, so it runs for the cleaned-up done row only.

2. **MEDIUM, design — no negative row for another run's gate or fixer.**
   - Location: `internal/battencli/wire_seams_test.go`, the added loop; behaviour at `internal/battencli/wire.go:699`.
   - Summary: the focus asks that another run's gate cannot hold this run's notice. Every added row registers under `taskLocation.WorktreeName`, and no row registers a gate or fixer record under a different slug and expects false. Swapping `runproc.List(dir, WorktreeName)` for `runproc.Owners(dir)` or a wrong owner would pass all tests. The test's own comment claims only "a process of another role" is ignored.
   - Evidence: the added loop body at `wire_seams_test.go:546-563`.

3. **LOW, design — no row for stamp write or read failure.**
   - Location: `internal/shuttleengine/activity.go:139` and `wait.go:1927`.
   - Summary: the "failure leaves the old fallback intact" behaviour is unpinned. `ReadNewestRunEnd` ignores a non-ENOENT `os.Stat` error on the stamp, and `finalize` only warns when `stampRunEnd` fails. No row has an unwritable root or a stamp that is a directory or unreadable.
   - Evidence: `activity_test.go` rows only vary `endStamp` through `stampRunEnd` on a good root.

4. **LOW, design — no row for a dead or recycled-pid registration in `gateProcessLive`.**
   - Location: `internal/battencli/wire_seams_test.go`.
   - Summary: the test registers only the live test process, so the dead and recycled-pid exclusion is not exercised through `readAgentActivity`. It may be covered by `runproc`'s own tests, which I did not read. Drop this finding if those pin pruning (a redundant test would be the defect). Raise it only if they do not.
   - Evidence: records are `runproc.SelfRecord(step.role)` only.

Tool calls:
1. `grep -n "waitLive" internal/dobbyengine/runnotice_test.go`: to check whether the existing dobby notice tests already pin a live wait holding off the quiet notice (they do, at line 230); the manifest did not include that test file.
