```yaml
lens: code-tests
outcome: reported
```

Per behaviour, the test that pins it:

- Report optional `journal` (absent / round-trip): `internal/runplan/report_test.go` `TestReport_JournalIsOptional`.
- Begun-card hash move (moved to runplan): `internal/runplan/fingerprint_test.go` `TestMoveBegunCardHashes`.
- Plan-conflict way forward from caller, by cause (incl. plan_order): `internal/websterengine/beginbatch_test.go` (matches "begin-batch %02d again", "move each named delete").
- Missing-index error text (`errNoIndex` now `runplan:`): `beginbatch_test.go` "no code index".
- FindBatch/AwaitBatch way forward from caller: `beginbatch_test.go` / `recordbatch_test.go` ("lists the run's batches").
- MapPartition way forward from caller: `internal/websterengine/partition_test.go:209-217`.
- Later-target hand-back way forward from caller: `internal/websterengine/recordbatch_git_test.go:196` ("lym webster record-batch 01").
- PostBatchChecks computing completed cards from `Batches` + `RecordWarning` callback: `recordbatch_git_test.go` / `recoverbatch_test.go` "later card:" warnings and hand-back.
- `CardNotDoneError` exported fields read by `failCardNotDone`: `recoverbatch_test.go:1434`, record-batch tests via `export_test.go`.
- Fingerprint/batch-card edit way forward from caller: `internal/websterengine/fingerprint_test.go` `TestPlanEditErrors`.
- Verify findings way forward from caller: `internal/websterengine/verifygatereport_test.go`, `verifygate_test.go`.
- Conflict record header kept byte-identical: `internal/webstercli/verbs_test.go:471`, `internal/shedadapters/webster_test.go:206`.
- Embedded `runplan.State` flattened into state.json: `internal/websterengine/state_test.go:136` `TestLoadState_UnusualFiles` loads a hand-written state.json with top-level `"batches"`, which only passes when the embedded struct is flattened.
- Suspect-path way forward from `PersistRecoveryTerminal` (`planPathClause`): **none** — see finding 1.
- Per-field JSON keys of moved run-plan fields (`planFingerprint`, `planFileHashes`, `planOverviewFrameHash`, `partition`): only typed round-trips (`TestState_RoundTrip`) — see finding 2.

Findings:

1. LOW, design — `internal/websterengine/recoverbatch.go:688` (`runplan.CheckRecoveredSuspects(..., planPathClause(...recover-batch %02d...))`). The production call site now passes webster's way forward for a plan-file suspect path, but the only test of that reason text (`internal/websterengine/suspect_test.go:187`, assertion at :384) calls the `checkRecoveredSuspects` wrapper in `internal/websterengine/export_test.go`, which rebuilds the same `planPathClause` argument itself. No test drives `PersistRecoveryTerminal` to a plan-path reason (grep for "still differs from the plan as the run recorded it" finds no test), so a wrong or empty way forward at the production caller would pass every test — the exact behaviour D11 requires ("Webster passes its own way forward") is uncovered.

2. LOW, design — `internal/runplan/state.go:1808-1822` (moved `State` json tags). The keys of the moved fields are pinned only by typed save/load round-trips (`internal/websterengine/state_test.go:41` `TestState_RoundTrip`, webstercli seeds) and `TestLoadState_UnusualFiles` (`state_test.go:136`) pins flattening through `"batches"` alone; a changed tag on `planFingerprint`, `planFileHashes`, `planOverviewFrameHash` or `partition` would pass every test while silently dropping that field from an existing on-disk state.json of an in-flight run, which D11's "Webster's persisted keys unchanged" requires; suggested: add those keys to the raw state.json fixture.

Tool calls:
1. Grep `_test.go` files for each changed way-forward string, state key and sentinel — context had the diff, not the test files asserting on them.
2. Grep record-batch/partition/suspect/recover-batch tests for the exact assertion lines and the raw-key / conflict-header fixtures — to confirm which assertions pin the caller-passed text.
3. Grep `state_test.go` raw-key handling and `checkRecoveredSuspects`/`PersistRecoveryTerminal` test callers — to settle findings 1 and 2.
