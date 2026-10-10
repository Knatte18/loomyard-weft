# Card 5 — recovery-sessions-are-run-writes

**Edit:**
- `internal/websterengine#BatchState`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#RunWrites`
- `internal/websterengine#loadRunWrites`
- `internal/websterengine#recordedSessions` -> `func recordedSessions(st *State) (master, recovery []string)`
- `internal/websterengine#writtenWorktreePaths`
- `internal/websterengine#ownTrackedPaths`
- `internal/websterengine/doc.go`
- `internal/websterengine/runwrites_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/websterengine/recoverbatch_git_test.go`

**Uses:**
- `internal/websterengine#uncommittedPathsBlock`
- `internal/websterengine#PlanReset`

**Intent:** Record every recovery strand's session on the batch and count its writes as the run's own (decision D3).
`BatchState` gains `RecoverySessions []string` (`json:"recoverySessions,omitempty"`): the session id of every recovery strand spawned for the batch, in spawn order.
`recoverSpawn` appends the spawned run's session id, read from the `RunState` that `shuttleengine.FindRun` returns, to the prior record's list, so a respawn carries the earlier sessions forward exactly as `Recoveries` is carried; `BeginBatch`'s re-begin carries the list onto its fresh record beside `Recoveries`.
`RunWrites` gains `Recoveries []shuttleengine.WriteEvent`, every recovery session's own writes, session by session, in transcript order.
`recordedSessions` returns the Master sessions (as today) and, separately, the distinct recovery sessions of every batch in batch-number order; `loadRunWrites` audits the Master sessions first, then the recovery sessions, putting a recovery session's `ParentWriteEvents` into `Recoveries` and any fork of it into `Forks`, and still audits each id once.
`writtenWorktreePaths` and `ownTrackedPaths` read all three slices, so a second recovery's "What the worktree holds" lists the first recovery's uncommitted files under "Written by this run", and a reset discards a tracked path a recovery session of the run wrote successfully without the dirt refusal; a failed write stays no evidence, and a path no session of the run wrote still refuses.
Update `doc.go`: the recovery section says the batch records every recovery session and that the uncommitted-paths block's write evidence covers Master, every fork and every recovery session; the reset section says the own paths are Master's, every fork's and every recovery session's successful writes; the `BatchState.Recoveries` paragraph names `RecoverySessions`.

Tests: extend `TestLoadRunWrites` (`runwrites_test.go`) with rows where a batch records recovery sessions, asserting they are audited after the Master sessions, once each, and that their parent writes land in `Recoveries`;
extend the `writtenWorktreePaths` case with a recovery-written path that is listed.
In `recoverbatch_test.go`, assert on the spawn tests that the fresh record carries the spawned session id and, on a respawn, the prior sessions before it.
In `beginbatch_test.go`, assert a re-begin carries `RecoverySessions`.
Add a case to `TestPlanReset` (`reset_integration_test.go`): a dirty tracked path a recovery session wrote successfully is an own path and the plan is returned, while a dirty tracked path no session wrote still refuses.
Add a case to `TestRecoverBatch_UncommittedPathsPrompt` (`recoverbatch_git_test.go`): a path the earlier recovery session wrote renders under "Written by this run".

**ImpactSummary:** One new persisted field on BatchState, one new slice on RunWrites and one unexported re-sign with a single caller; every reader of run writes (recovery prompt, reset, contract evidence) now sees recovery writes.
