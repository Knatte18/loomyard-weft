# Card 8 — Webster reads each session through its recorded engine

**Delete:**
- `internal/websterengine#AuditEngineFrom`
- `internal/websterengine/auditengine_test.go`

**Create:**
- `plan:internal/websterengine#AuditEngines` -> `type AuditEngines interface`
- `plan:internal/websterengine#sessionAuditEngine` -> `func sessionAuditEngine(engines AuditEngines, session, recorded string) (AuditEngine, error)`
- `plan:internal/websterengine#recordedSession` -> `type recordedSession struct`

**Edit:**
- `internal/websterengine#AuditEngine`
- `internal/websterengine#RunDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#RecordDeps`
- `internal/websterengine#ResetDeps`
- `internal/websterengine#recordedSessions` -> `func recordedSessions(st *State) (master, recovery []recordedSession)`
- `internal/websterengine#loadRunWrites` -> `func loadRunWrites(engines AuditEngines, st *State, worktree string) (RunWrites, error)`
- `internal/websterengine#contractWritesFor` -> `func contractWritesFor(engines AuditEngines, st *State, geom Geometry, paths []string) (RunWrites, error)`
- `internal/websterengine#pendingAuditFindingsError` -> `func pendingAuditFindingsError(engines AuditEngines, st *State, geom Geometry, reentry string) error`
- `internal/websterengine#pendingFindingsText` -> `func pendingFindingsText(engines AuditEngines, st *State, geom Geometry, items []findingItem, reentry string) (clause, wayForward string, err error)`
- `internal/websterengine#freshPendingDrop` -> `func freshPendingDrop(engines AuditEngines, geom Geometry, st *State, opts RunOptions) (drop bool, warnings []string, err error)`
- `internal/websterengine#checkPendingFindings` -> `func checkPendingFindings(engines AuditEngines, geom Geometry, st *State, base string, guard pendingGuard) ([]string, error)`
- `internal/websterengine#ArchiveRunAfterReset` -> `func ArchiveRunAfterReset(engines AuditEngines, geom Geometry, st *State) ([]string, error)`
- `internal/websterengine#AcceptPendingAudit` -> `func AcceptPendingAudit(engines AuditEngines, st *State, geom Geometry, parentBranch ParentBranchFunc) (accepted []PendingAuditFinding, onAbsentContract bool, err error)`
- `internal/websterengine#Run`
- `internal/websterengine#runExitAuditCrossCheck`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#auditTerminalFork`
- `internal/websterengine#uncommittedPathsBlock`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#awaitTerminal`
- `internal/websterengine#ownTrackedPaths`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.setRunner` -> `func (c *websterCLI) setRunner(runner *shuttleengine.Runner, engines shuttleengine.EngineSet, reedEngine shuttleengine.ReedOps)`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.runDeps`
- `internal/webstercli#websterCLI.recoverDeps`
- `internal/webstercli#websterCLI.recordBatchCmd`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/shedfake#shuttleEngine`
- `internal/testkit/shedfake#WebsterSeams`
- `internal/testkit/shedfake#TestWebsterSeams_FieldsNonNil`
- `internal/websterengine#TestLoadRunWrites`
- `internal/websterengine#TestPendingFindingsText`
- `internal/websterengine#TestAcceptPendingAudit_RefusesPathlessFinding`
- `internal/websterengine#TestAcceptPendingAudit_ContractFiles`
- `internal/websterengine#TestAcceptPendingAudit_RefusesMissingCommit`
- `internal/websterengine#TestAcceptPendingAudit_UsesExecutionOrderHead`
- `internal/websterengine#TestAcceptPendingAudit_ClearsWhenPathsMatchHead`
- `internal/websterengine#TestAcceptPendingAudit_RefusesDifferingPath`
- `internal/websterengine#TestAcceptPendingAudit_RefusesCommitPastHead`
- `internal/websterengine#TestAcceptPendingAudit_AcceptsAfterHeadReset`
- `internal/websterengine#TestAcceptPendingAudit_RefusesUnverifiablePath`
- `internal/websterengine#TestAcceptPendingAudit_PlanPathNamesRestorePlan`
- `internal/websterengine_test#TestArchiveRunAfterReset`
- `internal/websterengine_test#TestRun_DoneWithParentWriteToTrackedFileDemotesToStuck`
- `internal/websterengine_test#newRunFixtureOver`
- `internal/websterengine_test#newRecoverFixtureOver`
- `internal/websterengine_test#newRecordFixtureOver`
- `internal/websterengine_test#TestRecordBatch_AuditOutcomes`
- `internal/websterengine_test#TestParentMergeBetweenForkCommitAndRecordBatch`
- `internal/websterengine_test#TestRecoverBatch_SecondCall`
- `internal/websterengine#resetFixture.step`
- `internal/websterengine#TestPlanReset`
- `internal/webstercli#newVerbsFixture`
- `internal/webstercli#newResetFixture`
- `internal/webstercli#TestSmoke_RecordBatchConsumesCrashedSessionReport`
- `internal/loomcli#TestWire_DefaultConfig`
- `contracts/specs/refusal-spec.md`

**Intent:**
D6's read side.
`AuditEngines` is the lookup `webstercli` and loom's Webster wiring hand `websterengine` in place of one default `AuditEngine`: `Lookup(name string) (shuttleengine.Engine, error)`, an empty name resolving the default, and `DefaultName() string`.
`shuttleengine.EngineSet` satisfies it, so the wiring passes the engine set it already built, and `websterengine` resolves no engine set itself (`PATTERN-told-geometry`).
`RunDeps`, `RecoverDeps`, `RecordDeps` and `ResetDeps` replace `Engine` with `Engines AuditEngines`; a nil `Engines` keeps the nil-engine meanings each site has today (no write history, no reset own paths).
`AuditEngineFrom` is deleted, and neither `webstercli`'s two wiring prologues nor `loomCLI.wire` check any engine, since the wiring runs once per invocation for every verb; `websterCLI` holds the engine set, and `setRunner` takes it.

`sessionAuditEngine` resolves the engine of one session from the name its state entry records, three ways:
- a name the set holds is read through that engine; one without fork audit is unreachable (every spawn checks its engine, card 9) and returns an internal error naming the session and the engine, not a refusal row;
- a name the set does not hold (the `lyx` binary changed between spawn and read) is refused, wrapping the lookup's `ErrEngineNotRegistered` and naming the session and its engine, with the way forward `re-run the verb, or under the Webster row re-step the row, with a lyx whose engine set holds "<engine>"`;
- an empty name (an entry written before this change) reads as the current default, and is refused when that default has no fork audit, naming the session and `shuttle.yaml`'s `engine:` value, with the way forward `set engine: in shuttle.yaml back to an engine with fork audit, then re-run the verb, or under the Webster row re-step the row`.

The readers resolve each session's engine from its own state entry, so a later change to the `master` or `recovery` key never changes it:
`recordedSessions` returns each session id with its recorded engine (Master's id with `MasterEngine`, each batch's `SessionID` with its `Engine`, each recovery session with its `RecoveryEngines` entry, empty when absent), the first sighting of an id fixing its engine;
`loadRunWrites` audits each through `sessionAuditEngine`;
`RecordBatch` and `auditTerminalFork` audit `bs.SessionID` through the batch's `Engine`;
`awaitTerminal` reads the live recovery record's turn ends (`TurnEndedAfter` over its `EventsPath`) through the engine `RecoveryEngines` records for the last entry of its `RecoverySessions`, a missing key being an engine-less entry.
`contractWritesFor`, the pending-findings and `--fresh` paths, `AcceptPendingAudit`, `ArchiveRunAfterReset`, `uncommittedPathsBlock`, `RecoverSpawnOrAttach`'s contract-file check, `ownTrackedPaths` and `runExitAuditCrossCheck` pass `Engines` through.
`doc.go` describes the per-session resolution, and `auditengine.go`'s file header, which names the deleted assertion, is reworded to say the file declares the audit engine type and the lookup webster is told.

`contracts/specs/refusal-spec.md`'s webster section replaces the `engine without fork audit` row, whose wiring trigger is gone, with two rows: `session engine not registered` and `engine-less session on a default without fork audit`, each with the way forward above, class correctness halt (the role-key spawn check's row lands with card 9).

Tests, rows in `TestLoadRunWrites` over a `shuttleengine.NewEngineSet` of named fakes:
- a state whose Master session records engine `a` and whose batch records `b` audits each session through its own engine (each fake records the session ids it was asked for), while the set's default is a third engine;
- a batch with two recoveries, `r1` on `a` and `r2` on `b`, audits `r1` through `a` and `r2` through `b`, so after two recoveries, or a re-begin that carried them, the first recovery's session is read through its own engine;
- a session recording an engine the set does not hold is refused naming that engine, with `ErrEngineNotRegistered` wrapped;
- an engine-less entry while the default hides its fork audit (`struct{ shuttleengine.Engine }{fake}`) is refused naming `engine:` and its way forward;
- an engine-less entry while the default audits reads through the default;
- a session recording a held engine that hides its fork audit returns the internal error naming the session and the engine, with no way forward;
- an id recorded as Master's with `MasterEngine` `a` and again as a batch's `SessionID` with `Engine` `b` is audited once, through `a`, the first sighting;
- a nil `Engines` yields an empty history, as a nil engine does today.
`TestPlanReset` gains a row over a state whose batch's recovery session records engine `b` in `RecoveryEngines`, with no shuttle run directory for it on disk, while the set's default is `a`: the reset's own paths come from `b`'s fake, which is `reset` reading a recovery whose run directory was removed after its done run.
`TestRecordBatch_AuditOutcomes` gains rows:
- a batch records engine `b` while the set's default is `a`: the audit reaches `b`'s fake only, which stands for record-batch after the `master` key changed;
- a batch records an engine the set does not hold: `RecordBatch` refuses naming it with `ErrEngineNotRegistered` wrapped and archives no report, since the error is not `fs.ErrNotExist`;
- an engine-less batch while the default hides its fork audit is refused with the `engine:` way forward;
- a terminal fork batch whose record names engine `b` routes `auditTerminalFork`'s audit through `b`'s fake.
`TestRecoverBatch_SecondCall`'s rows that swapped `Deps.Engine` for a `claudeengine.New()` now register it in `Deps.Engines` under the name the live record's last recovery session records; further rows cover the turn-end read's other branches:
- the last session's `RecoveryEngines` key is absent and the default audits: read through the default;
- the last session records an engine the set does not hold: the call refuses naming it;
- the key is absent and the default hides its fork audit: the call refuses with the `engine:` way forward;
- a live record with an empty `RecoverySessions`, as written before this change, reads as an engine-less entry through the default.
The other listed tests and fixtures move from an engine argument or field to `shuttlefake.EngineSet(engine)`; `shedfake`'s placeholder becomes an `AuditEngines` stand-in.
`internal/websterengine/auditengine_test.go` held only `TestAuditEngineFrom` and goes with the function.

**ImpactSummary:** webster's audit seam changes from one engine to a lookup across websterengine's readers, webstercli's wiring and verbs, loomcli's wiring and shedfake; the wiring-level refusal is removed and its row replaced by two read-side rows.

**Commit:** 8: webster reads each session through its recorded engine
