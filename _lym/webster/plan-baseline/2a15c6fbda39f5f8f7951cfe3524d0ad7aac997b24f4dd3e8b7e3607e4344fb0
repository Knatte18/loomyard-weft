# Card 5 — usage-wiring

**Create:**
- `plan:internal/shedadapters#UsageRecorder` -> `func UsageRecorder(l *lymcwd.Location, readers map[string]shuttleengine.RecordReads) func(shuttleengine.SessionEnd)`
- `plan:internal/shedadapters#TestUsageRecorder` -> `func TestUsageRecorder(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestStatusCommitPathspec`
- `internal/gatecli#newStrandRig`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.primeShuttle`

**Uses:**
- `plan:internal/shedadapters#RecordSessionUsage`
- `plan:internal/shedrun#UsageDir`
- `plan:internal/shedrun#UsageDirRel`
- `internal/shedrun#StatusFile`
- `internal/shuttleengines#RecordReaders`
- `plan:internal/shuttleengine#Runner.SetSessionEnd`
- `internal/shedadapters/doc.go`
- `internal/gatecli/doc.go`

**Intent:** D12: every agent session a run spawns writes its usage file.
`UsageRecorder` returns the session-end hook of a run's worktree: it reads the run's current row, the `CurrentProducer` of the run's status file (`shedrun.StatusFile(l, shedrun.SelfRunID)`, an unreadable status giving an empty row), picks `readers[end.Engine]`, and calls `RecordSessionUsage(shedrun.UsageDir(l), row, end, reads)`.
The hook is installed with `runner.SetSessionEnd(shedadapters.UsageRecorder(<the run's worktree location>, shuttleengines.RecordReaders()))` on every runner a run's processes build:
- `loomCLI.wire`, beside `SetNotifier`, on the run's one runner, so the writers, reviewers, fixers, judges, Webster's master and its forks (recorded inside their master's file, as the transcript reader reads them), the gate fixers loom spawns and Sampler-Session's unawaited session through the stop verb are recorded;
- `loomCLI.stopStepRuns`' runner, so a session the step watchdog stops is recorded;
- `gatecli`'s `newStrandRig`, the runner of a card-gate fixer, the one session of a run started outside loom's process;
- `battenCLI.wire`'s runner over the task location, and `battenCLI.primeShuttle`'s runner of batten's driver and judge sessions, with the task worktree's location as `battenCLI.wire` resolves it, so batten's sessions over a task count toward that task's run.
Bound: the stops `lym batten cancel` and pair teardown make carry no hook; they end a run being cancelled or removed, which never lands, and `lym report cost` reads landed runs only (D13), so its cells lose nothing.
`statusCommitPathspec` appends `shedrun.UsageDirRel()` when the usage directory holds a file (`shedrun.HoldsFile`), the same rule the reviews directory follows, so each status commit carries the usage files written since the last one.
`TestUsageRecorder` (tier0) drives the hook over a temp location with a seeded status file: a session end writes `session-<id>.json` naming the status's current row; a status file that is absent writes the record with an empty row; an engine with no readers entry hands `RecordSessionUsage` an empty `RecordReads`, asserted by the written record's null cost alone, since card 4's test covers the counts.
`TestStatusCommitPathspec` gains a row with a usage file present (the usage directory joins the pathspec) and keeps its empty-directory row (omitted).

**ImpactSummary:** Five runner construction sites gain the hook and one commit pathspec grows by an optional directory; no producer or verb changes.
