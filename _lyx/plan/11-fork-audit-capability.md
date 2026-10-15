# Card 11 — fork-audit-capability

**Delete:**
- `internal/shuttleengine#Engine.AuditForks`
- `internal/shuttleengine#Engine.AuditForksIncremental`

**Create:**
- `plan:internal/websterengine#AuditEngine` -> `type AuditEngine interface`
- `plan:internal/websterengine#AuditEngineFrom` -> `func AuditEngineFrom(engines shuttleengine.EngineSet) (AuditEngine, error)`
- `internal/websterengine/auditengine_test.go`

**Edit:**
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#TestRun_Wait_ForkAuditFailure_KeepsTheClassifiedOutcome`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine#Claude.AuditForks`
- `internal/shuttleengine/claudeengine#Claude.AuditForksIncremental`
- `plan:internal/shuttleengines#build`
- `internal/shuttleengines/shuttleengines_test.go`
- `internal/websterengine#TestPendingFindingsText`
- `internal/websterengine#TestAcceptPendingAudit_ContractFiles`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine#RunDeps`
- `internal/websterengine#ResetDeps`
- `internal/websterengine#RecordDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#loadRunWrites` -> `func loadRunWrites(engine shuttleengine.ForkAuditor, st *State, worktree string) (RunWrites, error)`
- `internal/websterengine#contractWritesFor` -> `func contractWritesFor(engine shuttleengine.ForkAuditor, st *State, geom Geometry, paths []string) (RunWrites, error)`
- `internal/websterengine#checkPendingFindings` -> `func checkPendingFindings(engine shuttleengine.ForkAuditor, geom Geometry, st *State, base string, guard pendingGuard) ([]string, error)`
- `internal/websterengine#ArchiveRunAfterReset` -> `func ArchiveRunAfterReset(engine shuttleengine.ForkAuditor, geom Geometry, st *State) ([]string, error)`
- `internal/websterengine#AcceptPendingAudit` -> `func AcceptPendingAudit(engine shuttleengine.ForkAuditor, st *State, geom Geometry, parentBranch ParentBranchFunc) (accepted []PendingAuditFinding, onAbsentContract bool, err error)`
- `internal/websterengine#pendingAuditFindingsError` -> `func pendingAuditFindingsError(engine shuttleengine.ForkAuditor, st *State, geom Geometry, reentry string) error`
- `internal/websterengine#pendingFindingsText` -> `func pendingFindingsText(engine shuttleengine.ForkAuditor, st *State, geom Geometry, items []findingItem, reentry string) (clause, wayForward string, err error)`
- `internal/websterengine#freshPendingDrop` -> `func freshPendingDrop(engine shuttleengine.ForkAuditor, geom Geometry, st *State, opts RunOptions) (drop bool, warnings []string, err error)`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#auditTerminalFork`
- `internal/websterengine#ownTrackedPaths`
- `internal/websterengine#uncommittedPathsBlock`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#Run`
- `internal/websterengine#runExitAuditCrossCheck`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/archivereset_test.go`
- `internal/websterengine/runwrites_test.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.setRunner` -> `func (c *websterCLI) setRunner(runner *shuttleengine.Runner, auditEngine websterengine.AuditEngine, reedEngine shuttleengine.ReedOps)`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/shedfake#shuttleEngine`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Move `AuditForks` and `AuditForksIncremental` off `Engine` into the optional `ForkAuditor` capability card 3 declared (D7).

Shuttle: `Engine` drops both methods.
`Run.finalize` takes the audit of a `ForkSubagents` record that classified done from `run.engine.(ForkAuditor)`.
When the run's engine lacks the capability, `finalize` returns the classified result with a nil `ForkAudit` and the error `shuttle: run <run dir>: engine "<engineName>" cannot audit the forks its ForkSubagents record authorizes`, the path an audit error takes today, so the run never finishes without its audit and its directory is kept.
`Claude.AuditForks` and `Claude.AuditForksIncremental` say in their doc comments that they implement `shuttleengine.ForkAuditor`, and `recordbatch_test.go`'s file header names `ForkAuditor.AuditForksIncremental`.
`internal/shuttleengine/doc.go` names fork audit among the optional capabilities.

D8's registry guard lands here, now that an `Engine` no longer satisfies `ForkAuditor` by itself: `shuttleengines`' `build` refuses an engine that declares `RequirementForks` through `RequirementSupporter` without implementing `ForkAuditor`, with `shuttle: engine "<name>" declares forks but cannot audit them`, a wiring guard no shipped engine reaches.
`TestNew` in `shuttleengines_test.go` gains a row: `build` over a constructor map holding a test engine whose `Supports(RequirementForks)` is true and that has no `ForkAuditor` is refused naming that engine.

Webster takes the capability wherever it takes an `Engine` for audits:
- `RunDeps.Engine`, `ResetDeps.Engine` and `RecordDeps.Engine` become `shuttleengine.ForkAuditor`.
- `RecoverDeps.Engine` becomes `AuditEngine`, a new interface in `internal/websterengine/auditengine.go` embedding `shuttleengine.Engine` and `shuttleengine.ForkAuditor`, since the recover path also parses the batch's events through `TurnEndedAfter`, which keeps taking an `Engine`.
- The eight audit helpers take `shuttleengine.ForkAuditor`; their in-package callers pass the same values they pass today, which satisfy it.
  `TestPendingFindingsText` and `TestAcceptPendingAudit_ContractFiles` hold fixture funcs whose result type is `shuttleengine.Engine`, which no longer satisfies `ForkAuditor`; their result type becomes `shuttleengine.ForkAuditor`.
- `AuditEngineFrom(engines)` resolves the set's default engine and asserts both interfaces.
  A default that does not resolve returns the lookup's error, a fail-loud wiring guard with no per-row test: `shuttleengines.New` refuses a `cfg.Engine` naming no registered engine (card 5's `build`), so no production input reaches it, and card 1's `TestEngineSet_Lookup` already pins the lookup's error.
  An engine without fork audit is a webster refusal: `webster: shuttle.yaml's engine: "<name>" cannot audit forks, which webster needs; way forward: set engine: in shuttle.yaml to an engine with fork support, then re-run the verb, or under the Webster row re-step the row`.
  The input that reaches it is a `shuttle.yaml` whose `engine:` names a registered engine without `ForkAuditor`; `shuttleengines.New` already refuses an unregistered name, and no engine registered today lacks the capability.

Both wirings that hand Webster its engine assert it through `AuditEngineFrom` and return its error: `websterCLI.wireHub` and `wireStandalone` pass the result to `setRunner`, whose `websterCLI.engine` field becomes `websterengine.AuditEngine`, which `acceptAuditCmd` and `resetCmd` pass on as today; `loomCLI.wire` sets `RunDeps.Engine` from it.
`shedfake`'s `shuttleEngine` placeholder embeds `shuttleengine.ForkAuditor` instead of `shuttleengine.Engine`.

`contracts/specs/refusal-spec.md`'s webster section gains the row `engine without fork audit`: a webster verb's wiring, or loom's wiring of the Webster row, finds `shuttle.yaml`'s `engine:` naming a registered engine without fork audit; correctness halt; the way forward clause above (`PATTERN-refusal-way-forward`).

Tests:
- `TestRun_Wait_ForkAuditFailure_KeepsTheClassifiedOutcome` gains a row beside its audit-error row: a `ForkSubagents` record over a runner whose engine is `struct{ Engine }` wrapping a `fakeEngine`, which hides its audit methods, finalizes done with the error naming the engine, a nil `ForkAudit` and its run directory kept.
- `auditengine_test.go` (package `websterengine_test`) adds `TestAuditEngineFrom`, table-driven over `shuttlefake.EngineSet`: a `*shuttlefake.Engine` comes back as the `AuditEngine`; `struct{ shuttleengine.Engine }` wrapping one is refused with the exact refusal text naming `fake` and the way forward.
Raise `internal/websterengine` in `test-budget.yaml` from 202 to 203.

**ImpactSummary:** Drops two methods from the Engine interface; the fork audit's shuttle caller, the registry's forks guard, eight websterengine helpers with their callers and two test fixtures, four deps structs, both Webster wirings, one shedfake placeholder, two Claude doc comments and one refusal row follow.
