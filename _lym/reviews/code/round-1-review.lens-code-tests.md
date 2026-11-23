```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, which finds the test row that pins each behaviour the diff adds or changes.

**Per behaviour, the test that pins it**

1. `ClassifyHalt` returns `AnomalyBuildSkew` for a failed, uninterrupted halt with `buildSkew` set: pinned by `TestClassifyHalt` row `FailedBuildSkew_BuildSkew` (`internal/loomengine/anomaly_test.go`, the added row near line 161).
2. An interrupted stop wins over `buildSkew`: pinned by `TestClassifyHalt` row `FailedInterruptedBuildSkew_InterruptedStop` (same file, the added row near line 169).
3. Without the flag, a failed halt is still `AnomalyProducerFailure`: pinned by the existing `TestClassifyHalt` failed row (`anomaly_test.go:158`) and by `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory` row `Failed` (`internal/loomcli/halt_test.go:175`).
4. The build-skew case applies only to a failed state, through the `state == shedengine.StateFailed && buildSkew` guard (`internal/loomengine/anomaly.go:107`): **no row.** No row passes `buildSkew: true` with a blocked, running, done or other non-failed state, so deleting the state guard passes the whole table. See finding F1.
5. `failedHalt` sets `BuildSkew` when a `*stencilstore.NewerBoardCopyError` is in the step error's chain, and `writeHaltNote` passes the flag on: pinned end to end from an error value by `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` row `BoardStencilFromANewerBuild` (`halt_test.go`, added row near line 478).
   - The producer returns the skew error wrapped with `%w` behind the real `shedadapters` prefix.
   - shedengine persists it, and the note must contain `anomaly: build-skew\n`.
   - In production, a transient mark keeps the chain, since `transientError` has `Unwrap` (`internal/shedengine/transient.go:41`).
6. A step error whose chain lacks the skew type stays `producer-hard-failure` in the note: pinned by the same test's row `ProducerFailure`, which asserts `anomaly: producer-hard-failure\n`.
7. The `loomPostRun` (`lym loom run`) path reuses `failedHalt`, so it is pinned at the function by behaviour 5. No run-path test drives a skew error, and none is needed: there is no separate branch.
8. Static prose in `friction-template-reflection.md` and `loomcli/doc.go` is exempt.

**Assertions narrowed or removed.** None. The old check that the note contains "producer exploded" stays as `tt.err.Error()`, the error envelope, friction-skipped and zero-reflection assertions are kept for every row, and the new anomaly-line check is stricter.

**Findings**

- **F1.** Severity: LOW. Class: design. Location: `internal/loomengine/anomaly_test.go`, `TestClassifyHalt` table.
  - Summary: the new state guard on the build-skew case has no row. A blocked or non-failed state with `buildSkew: true` must keep its own kind: escalation or budget-exhausted for blocked, no kind for running or done. Dropping `state == shedengine.StateFailed &&` at `anomaly.go:107` passes every row.
  - Evidence: `internal/loomengine/anomaly.go:107-108`, and the table rows in the diff piece `003-internal-loomengine.diff`, none of which sets `buildSkew` outside `StateFailed`.
  - Fix: add a `TestClassifyHalt` row with a non-failed state, for example `StateRunning` or `StateDone`, `buildSkew: true`, and want `("", false)`.

Tool calls:
1. Ran grep over `internal/shedengine/transient.go`, `internal/loomcli/halt_test.go` and the `internal/loomcli` production files. My context lacked two facts: whether `transientError` keeps the error chain (its `Unwrap`), and whether the halt fixture or `loomcli` production sets a transient classifier that could re-wrap the skew error.
