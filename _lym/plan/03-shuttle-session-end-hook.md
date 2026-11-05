# Card 3 — shuttle-session-end-hook

**Create:**
- `internal/shuttleengine#SessionEnd`
- `internal/shuttleengine#Runner.SetSessionEnd`
- `internal/shuttleengine#TestRunner_SessionEnd`

**Edit:**
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Runner.launch`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#Runner.stopRecorded`
- `internal/shuttleengine#TestRun_Finalize_ReportsUsageAndTimes`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#usageEngine`
- `internal/shuttleengine#Runner.StopStrand`
- `internal/shuttleengine#UsageReader`

**Intent:** The seam D12's writer hangs on, general to every caller of shuttle, with nothing of shed in it (`PATTERN-fabric-isolation` in spirit: shuttle learns no run, row or board).
`SessionEnd` carries one ended session: `SessionID`, `Engine` (the engine name), `Role` (the spec's role segment), `Model` (the spec's model), `Worktree` (the pane cwd the provider's transcripts derive from), `StartedAt`, `EndedAt`, `Outcome` and `Usage SessionUsage`.
`RunState` gains `Role` and `Model` (`omitempty`), which `Runner.launch` records in `run.json` at the start, so a path holding only the record, the stop path, can fill them; a record an older binary wrote leaves them empty.
`Runner` gains a `sessionEnd func(SessionEnd)` field set by `SetSessionEnd`; nil, the default, calls nothing.
`Run.finalize` calls it once per terminal outcome, after the usage read of card 1 and before the cleanup, with the result's and spec's fields; a session with no session id (one that never started) is not reported.
`Runner.stopRecorded`, when it moves a running record to stopped, also calls it, with the record's session id, engine, `Role`, `Model`, `CreatedAt` as the start, the stop time as the end, outcome stopped, and the usage the record's engine (resolved through the runner's engine set) reads through `UsageReader` for the session id and `paneCwd`, the zero usage when the engine has none; so an unawaited session the stop verb ends, such as Sampler-Session's, is reported too.
A record already terminal is not reported again, since its own finalize reported it.
The hook runs synchronously and its panics are not recovered; its contract, stated on `SetSessionEnd`, is that it never blocks and reports its own failures.
`TestRun_Finalize_ReportsUsageAndTimes` gains a hook on its runner and asserts, in each of its done, stuck and died rows, one call carrying the outcome, role, model, times and usage, and, in a row whose run never started, no call.
`TestRunner_SessionEnd` (tier0) has rows: a `StopStrand` of a running unawaited record, written on disk naming the `usageEngine` fake's engine name with a role and model, on a runner whose engine set resolves that name, calls the hook once with outcome stopped and the record's role, model and the fake's usage; a stop of a terminal record calls it not at all.
`internal/shuttleengine/doc.go` gains one paragraph on the session-end hook.

**ImpactSummary:** One new optional runner field read at two terminal sites and two record fields written at launch; every existing runner leaves the hook nil, so no current caller changes behaviour.
