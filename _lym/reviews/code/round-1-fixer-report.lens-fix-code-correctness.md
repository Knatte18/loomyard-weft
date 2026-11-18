```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the fixer's change set.

1. **MEDIUM** — `internal/reedengine/doc.go`, the added lines "Reed never clears a hold on its own, so a caller whose release failed removes the strand when it gives the run up."
   - The claim is not what the code does, and the F3 premise stays open. After `ReleaseStrandHold` fails, `stopForRelaunch` (`internal/shuttleengine/stop.go`) returns an error wrapping both failures.
   - `relaunchUnderAccount` wraps it in `errRelaunchDeclined`. `switchAccountAtLimit` then returns `handled=false` and Wait carries on, so the run is not given up and nothing removes the strand.
   - The strand stays live and held, so reed's resume skips it if it later dies. The only change is that the release failure now appears in the declined warning.
   - The new test `TestRun_StopForRelaunch_ReturnsAFailedHoldReleaseBesideTheEndFailure` pins only the error text, not any clearing of the hold.
   - Either make the declined path remove or retry the release when the release failed, or reword the doc to say who owns the hold without promising removal.

2. **LOW** — `TestRun_SwitchAccountAtLimit_ADeclinedRelaunchHandlesTheLimitAsWithoutASwitch` in `accountrelaunch_test.go` relies on `permkit.Chmod(runDir, 0o555)` to make the mark write fail.
   - Under a root user, or on Windows, the chmod does not block writes. `switchAccountAtLimit` would then relaunch, `handled` would be true, and the test would fail rather than skip.
   - `stop_test.go`'s `unwritable_record` row has the same weakness, so this follows an existing convention.

3. **LOW** — `stopForRelaunch` now returns `errors.Join(endErr, "end its hold: …")` wrapped by `%w`.
   - The message therefore contains a newline, which makes the declined-relaunch warning log line multi-line. This is cosmetic.

I ran these through each function and found no defect:
- `readRegistry` with a missing directory: the three registry readers and their callers all handle the empty result.
- `validateAccounts` with empty home: it refuses before the `Join`.
- `turnEndsBetween` and its seeding: they leave `reseedAfterRelaunch` and the existing refused-resume rows unchanged.
- `checkRecordedBeforeRelaunch` and `recordAtRelaunch`: they fail if the record write moves after `RelaunchStrand`, because `Started` and the offsets would still read the old values.
- The F4 `promptOffset` row: it fails with the old per-Wait count.
- The F9 rows: the orphan within-bound row fails under the old pid-only liveness.
- The F12 lock probe: it fails if the names call runs outside the lock.
- The F13 env and newest-known-copy rows: each fails when its source is removed.

Tool calls used: none, since my context already held every changed file.
