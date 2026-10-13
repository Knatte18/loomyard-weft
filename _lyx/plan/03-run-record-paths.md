# Card 3 — run-record-paths

**Create:**
- `internal/shedrun#DriverLock`
- `internal/shedrun#TacklerDir`
- `internal/shedrun#TacklerRel`
- `internal/shedrun#TacklerAttemptDir`
- `internal/shedrun#TacklerMarker`
- `internal/shedrun#SettleRecordsFile`
- `internal/loomengine#TacklerRole`
- `internal/loomengine#TacklerVerdictResume`
- `internal/loomengine#TacklerVerdictHandBack`
- `internal/loomengine#TacklerVerdictEscalate`
- `internal/loomengine#WorktreeSnapshot`
- `internal/loomengine#WorktreeSnapshot.Equal`
- `internal/loomengine#TacklerAttempt`
- `internal/loomengine#WriteTacklerAttempt`
- `internal/loomengine#ReadTacklerAttempt`
- `internal/loomengine#ListTacklerAttempts`
- `internal/loomengine#TacklerVerdict`
- `internal/loomengine#ReadTacklerVerdict`
- `internal/loomengine#TacklerEnvelope`
- `internal/loomengine#WriteTacklerEnvelope`
- `internal/loomengine#ReadTacklerEnvelope`
- `internal/loomengine#TacklerPromptPath`
- `internal/loomengine#TacklerVerdictPath`
- `internal/loomengine#TacklerCaseBriefPath`
- `internal/loomengine#TacklerEnvelopePath`
- `internal/loomengine#MarkTacklerApplied`
- `internal/loomengine#TacklerApplied`
- `internal/loomengine#TacklerMarker`
- `internal/loomengine#WriteTacklerMarker`
- `internal/loomengine#ReadTacklerMarker`
- `internal/loomengine#RemoveTacklerMarker`
- `internal/loomengine#SettleRecord`
- `internal/loomengine#SettlerGoRule`
- `internal/loomengine#SettlerTackler`
- `internal/loomengine#SettlerArbiter`
- `internal/loomengine#SettlerParent`
- `internal/loomengine#SettleRestep`
- `internal/loomengine#SettleVerifyRetry`
- `internal/loomengine#SettleHandOver`
- `internal/loomengine#SettleSpawn`
- `internal/loomengine#SettleHandBack`
- `internal/loomengine#SettleBusyExit`
- `internal/loomengine#SettleRunningExit`
- `internal/loomengine#SettleMoved`
- `internal/loomengine#AppendSettleRecord`
- `internal/loomengine#ReadSettleRecords`
- `internal/loomengine#GoVerifyRetrySpent`
- `internal/loomengine/tackler_test.go`
- `internal/loomengine/settle_test.go`

**Edit:**
- `internal/shedrun/doc.go`
- `internal/shedrun#TestPathConstructors`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestStatusCommitPathspec`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
The records the Go driver and the tackler keep, declared where `PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state` put them.

`internal/shedrun/paths.go` gains, beside the arbiter constructors:
- `DriverLock`: the ephemeral Go-driver lock, `driver.lock` under `ScratchDir(l, runID)`, distinct from `RunLock` and `StatusLock`.
- `TacklerDir` and `TacklerRel`: the durable `tackler` directory under `RunDir`, and its anchor-relative form for a records pathspec.
- `TacklerAttemptDir`: `attempt-<N>` under `TacklerDir`.
- `TacklerMarker`: the ephemeral in-flight marker `tackler-inflight.json` under `ScratchDir`.
- `SettleRecordsFile`: the told friction directory joined with `go-settle.jsonl`.
  The name is not `.md`, so `frictionengine`'s note scan never archives it and the caps counted from it survive a reflection.
`internal/shedrun/doc.go` names the five paths and the settle file.

A new file `internal/loomengine/tackler.go` declares the tackler's records the way `arbiter.go` declares the arbiter's, reusing `writeArbiterJSON` and `readArbiterJSON`:
- `TacklerRole = "tackler"`;
- the verdict words `TacklerVerdictResume`, `TacklerVerdictHandBack` (`hand-back`) and `TacklerVerdictEscalate`;
- `WorktreeSnapshot` (`Head`, `Branch`, `Dirty map[string]string` from path to content hash) and its `Equal` method;
- `TacklerAttempt` (`Number`, `StopKey ArbiterStopKey`, `PauseBefore`, `PauseAfter`, `Snapshot WorktreeSnapshot`, `SpawnedAt`, `Deadline`, `EnvelopePath`, `ReportPath`, `Dir`, `StrandGUID`) with `WriteTacklerAttempt`, `ReadTacklerAttempt` and `ListTacklerAttempts`, the last sorted by number and counting from disk;
- `TacklerVerdict` (`verdict`, `reason`, `goto`, `question` in YAML) and `ReadTacklerVerdict(path) (TacklerVerdict, bool, error)`, which reports a file that does not decode, or carries a verdict word outside the three, as an error;
- `TacklerEnvelope` (`Verdict`, `Reason`, `Question`, `Goto`, `Cause`, `Attempt`, `AttemptDir`, `VerdictPath`, `CaseBriefPath`) with its writer and reader;
- `TacklerPromptPath`, `TacklerVerdictPath`, `TacklerCaseBriefPath` (`case-brief.md`) and `TacklerEnvelopePath` over an attempt directory;
- `MarkTacklerApplied` and `TacklerApplied` over the attempt directory's `applied` mark;
- `TacklerMarker` (`Attempt`, `StrandGUID`, `Deadline`) with `WriteTacklerMarker`, `ReadTacklerMarker` and `RemoveTacklerMarker`.

A new file `internal/loomengine/settle.go` declares the Go driver's settle records (D9):
- `SettleRecord` (`StopKey ArbiterStopKey`, `Settler`, `Action`, `Outcome`, `At time.Time`, `Build`, `Target`, the last two omitted when empty);
- the settler words `SettlerGoRule` (`go-rule`), `SettlerTackler`, `SettlerArbiter` and `SettlerParent`;
- the action words `SettleRestep`, `SettleVerifyRetry`, `SettleHandOver`, `SettleSpawn`, `SettleHandBack`, `SettleBusyExit`, `SettleRunningExit` and `SettleMoved`;
- `AppendSettleRecord(path string, r SettleRecord) error`, which creates the directory and appends one JSON line;
- `ReadSettleRecords(path string) ([]SettleRecord, error)`, which returns nil for an absent file, accepts unknown keys (`PATTERN-tolerant-records`), and skips a line that does not decode with one Warn, so a torn append never blocks a cap;
- `GoVerifyRetrySpent(records []SettleRecord) bool`, true when any record's action is `SettleVerifyRetry`, the one read batten's publish retry and the Go driver share (D4).

`statusCommitPathspec` stages `shedrun.TacklerRel` when `holdsFile(shedrun.TacklerDir(...))`, beside the arbiter directory (D7).
The settle file sits under the loom durable directory, which the pathspec already stages.

Tests:
- `TestPathConstructors` gains rows for `DriverLock`, `TacklerDir`, `TacklerRel`, `TacklerAttemptDir` and `TacklerMarker`, and `SettleRecordsFile` over a told directory.
- `TestStatusCommitPathspec` gains a row with a file under the tackler directory, expecting `TacklerRel` in the pathspec, and keeps an empty tackler directory out.
- `tackler_test.go` holds `TestTacklerRecords`: `WorktreeSnapshot.Equal` rows, where an equal snapshot is equal and a moved HEAD, a commit, a switched branch, a changed dirty-path set and an edited already-dirty path each are not (D7's tamper cases); an attempt, envelope and marker round-trip, `ListTacklerAttempts` orders and counts, the applied mark, and `ReadTacklerVerdict` on a valid file, an absent file, an unknown verdict word and undecodable YAML.
- `settle_test.go` holds `TestSettleRecords`: two appends read back in order, an absent file reads nil, an unknown key is kept readable, a torn last line is skipped, and `GoVerifyRetrySpent` is true only with a `verify-retry` record.
`test-budget.yaml`'s `internal/loomengine` row rises by two.

**ImpactSummary:** new constructors in `internal/shedrun` and new record files in `internal/loomengine`; the only existing behavior change is the status commit staging a non-empty tackler directory.
