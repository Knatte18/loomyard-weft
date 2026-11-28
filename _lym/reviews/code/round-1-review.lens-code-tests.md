```yaml
lens: code-tests
outcome: reported
```

Restating the task: the code-tests lens, which finds the test row that pins each behaviour the diff adds or changes.

**Behaviours and the tests that pin them**

1. **`TierSummaryLine` prints `wait <s>s` before `wall <s>s`.** Pinned by `TestTierSummaryLine/WaitAndWallApart` (`internal/gateslot/runlog_test.go:252`).
2. **`RunRecord.SlotWaitSeconds` returns 0 when the record has no wait start, and `Start − WaitStart` otherwise.** Pinned by `TestTierSummaryLine/NoWaitStartWaitedNone` (runlog_test.go:257) and `WaitAndWallApart`.
3. **`tierSummaryPattern` and `FailedTier` read the new line.** Pinned by `testFailedTier` (runlog_test.go:161-171, which builds its lines with `TierSummaryLine`) and by the `FailedTier` assertion in `TestTierSummaryLine`.
4. **`tierDiscoveredPattern` and `TierDiscovered` read the new line.** Pinned by `TestTierDiscovered` (runlog_test.go:210) and the `TierDiscovered` assertion in `TestTierSummaryLine`.
5. **`lym gate report` takes `wait_seconds` and `abandoned_wait_seconds` from `SlotWaitSeconds`.** Pinned by the `runReport` test rows at `internal/gatecli/report_test.go:27,31,35,36`, which carry a non-zero wait and assert `wait_seconds` and `abandoned_wait` (report_test.go:44,51).
6. **The wait figure the gate prints on each path** (`report` setting `WaitStart` from `runWithSlots`, a rerun zeroing it, a carried tier, a shared-build tier, `--copies`). No test pins it. `internal/gatecli/test_integration_test.go:904` checks only the carried line's prefix, up to `skipped 0`, and the "--wait blocks until a slot frees" subtest (test_integration_test.go:1396) asserts no wait figure.
7. **The help and package-doc prose.** Exempt as static prose.

**Findings**

- **MEDIUM**, class `design`, at `internal/gatecli/run.go:729-738` (`report`).
  No test pins the wait figure the gate prints for a real tier run.
  `TestTierSummaryLine` checks only how a hand-built record renders.
  It does not check how `WaitStart` and `Start` are set on each path: a slotted run, a rerun (run.go:731-732), a carried tier (run.go:663), a shared-build tier where `runBuild` runs between `waitStart` and the tier's `Start` (run.go:640-647, build.go:41,72, tierrun.go:130), or `--copies` with its lym prebuild before the copies start (copies.go:64-79).
  A wrong derivation on any of these passes every test.
  Evidence: test_integration_test.go:904 and 1396 assert no wait, and no gatecli test matches `wait [0-9.]*s` (grep found only runlog_test.go).
- **LOW**, class `consistency`, at `internal/gateslot/runlog_test.go:268-273`.
  The `FailedTier` and `TierDiscovered` assertions in `TestTierSummaryLine` repeat coverage that already exists.
  `testFailedTier` (runlog_test.go:161-171) and `TestTierDiscovered` (runlog_test.go:205-210) already pass `TierSummaryLine` output through those readers, so they read the new format.
- **LOW**, class `consistency`, at `internal/gateslot/startedlines_test.go:36`.
  The OtherLinesAreSkipped fixture still spells the summary line without `wait`, so it no longer shows the line the gate prints.
  The row still passes, since it only checks that the line is skipped, but it would pass unchanged if the gate printed the old format again.

**Tool calls**

1. Grepped gatecli tests for `WaitStart` and `wait_seconds`, and all tests for spelled summary lines: the manifest did not cover report_test.go or other literal fixtures.
2. Grepped gatecli and landingshed tests for summary-line assertions and rerun wait: to find any test that pins the printed wait.
3. Grepped `test_integration_test.go` for its summary-line and wait assertions: to see whether the integration tests assert the printed wait.
