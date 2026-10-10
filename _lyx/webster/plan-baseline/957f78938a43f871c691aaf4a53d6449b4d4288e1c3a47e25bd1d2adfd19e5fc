# Card 1 — verify-log-per-run

**Create:**
- `plan:internal/verifytree#verifyLogKeep` -> `const verifyLogKeep = 20`
- `plan:internal/verifytree#nextLogPath` -> `func nextLogPath(dir string) (string, error)`
- `plan:internal/verifytree#pruneLogs` -> `func pruneLogs(p Paths) error`
- `plan:internal/verifytree#IsLogPath` -> `func IsLogPath(p Paths, path string) bool`

**Edit:**
- `internal/verifytree#Paths`
- `internal/verifytree#Result`
- `internal/verifytree#NewPaths`
- `internal/verifytree#Verify`
- `internal/verifytree#PublishFailure`
- `internal/verifytree#WritePublishFailure`
- `internal/verifytree#RemovePublishFailure`
- `internal/verifytree/doc.go`
- `internal/verifytree/failure_test.go`
- `internal/verifytree/verify_integration_test.go`
- `internal/loomshed#NewVerifyGate`
- `internal/loomshed#checkedPublishFailure`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/loomshed/publishfailure_test.go`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#verifyGate.verdict`
- `internal/landingshed/verifygate_test.go`
- `internal/landingshed/verifygate_integration_test.go`
- `internal/landingshed/publish_verify_test.go`
- `internal/landingshed/finalize_verify_test.go`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#verifyGateSeams`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine/verifygate_test.go`
- `internal/webstercli#websterCLI.runVerify`
- `internal/webstercli/verify_integration_test.go`
- `pattern/PATTERN-verified-tree.md`

**Delete:**
- `internal/verifytree#logName`
- `internal/verifytree#publishFailureLogName`

**Intent:** D8: one verify log per run, named by sequence, with a bounded history, so a later verify never overwrites the evidence of an earlier failure (#502).
`Paths` drops `Log` and `PublishFailureLog` and gains `Dir`, the verify directory, which `NewPaths` fills from its `dir` argument.
`Verify` names each run's log `verify-<n>.log` in that directory through `nextLogPath`, `n` one above the highest existing number, and reports the path on `Result.Log`;
a dirty or skipped result carries an empty `Log`.
After the log is written, whatever the command's exit, `Verify` calls `pruneLogs`, which removes every log beyond the newest `verifyLogKeep` unless the Publish failure record names it;
a prune failure is logged and never changes the result.
`WritePublishFailure` no longer copies the log: it writes `f` as given, and the caller sets `f.LogPath` to the failing run's own `Result.Log`;
`RemovePublishFailure` removes the record alone.
`IsLogPath` reports whether `path` is a `verify-<n>.log` inside `p.Dir`, the shape check `checkedPublishFailure` applies to the record's `LogPath` in place of the equality against the retired copy.
Every reader of the retired `Paths.Log` reads the result's `Log` instead: `NewVerifyGate`'s failure findings, the landing gate's `recordFailure` (which also sets the record's `LogPath` from the result) and `verdict`, and webster's `runVerify`.
Webster's gate reruns once on a failure, so its seams read the log of the result they report on: `verifyGateSeams` drops the fixed `logPath` field, its `readLog` becomes `func(path string) (string, error)`, `websterengine.NewVerifyGate` fills it with a plain file read, and `newVerifyGate` passes the reported result's `Log` to `readLog` and into every report's `LogPath`.
The two retired name constants go.
Tests: the verifytree scenario gains steps asserting two consecutive verifies leave two logs each named by its own result, that the twenty-first log prunes the oldest, and that a log the failure record names survives pruning;
the failure-record test asserts the record keeps the caller's `LogPath` and that removal leaves the log in place;
every landing, loomshed and webster test that read `paths.Log` reads the fake or real result's `Log`, and the websterengine gate test's fake seams drop `logPath` and return a per-result `Log`, asserting a report names the log of the run it reports on.
`doc.go` and the verified-tree background drop the "copy of the log" clause and describe the per-run log and the retention bound.

**ImpactSummary:** Reshapes `verifytree.Paths` and `Result`; every verify site and failure-record reader in loomshed, landingshed, websterengine and webstercli follows, with their tests; no behavior outside the log file names changes.
