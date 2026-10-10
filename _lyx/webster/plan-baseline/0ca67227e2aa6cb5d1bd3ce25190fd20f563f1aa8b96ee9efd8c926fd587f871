# Card 15 — fabric-reference-policy-at-record-batch

**Edit:**
- `internal/websterengine#ClassifyViolation` -> `func ClassifyViolation(v AuditViolation, geom Geometry, readOnly func(cmd string) bool) (AuditSeverity, error)`
- `internal/websterengine#RecordDeps`
- `internal/websterengine#RunDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#auditTerminalFork`
- `internal/websterengine#runExitAuditCrossCheck`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.runDeps`
- `internal/webstercli#websterCLI.recordBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/wiring_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `//contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#IsReadOnlyCommand`

**Intent:** Class a fabric reference whose command is read-only as policy, so the batch records an audit warning and passes (D8).
`ClassifyViolation` takes the read-only classifier by injection, as `AcceptBatchFabricReference` already does, so websterengine keeps importing no `fabricengine`: a `fabric-reference` whose `Command` the classifier accepts is policy, every other fabric reference stays correctness, and a nil classifier accepts nothing.
`RecordDeps`, `RunDeps` and `RecoverDeps` gain `ReadOnly func(cmd string) bool`, and `RecordBatch`, `auditTerminalFork` and `runExitAuditCrossCheck` pass theirs through.
`webstercli` supplies `fabricengine.IsReadOnlyCommand` on all three: `runDeps`, record-batch's deps and recover-batch's deps.
The loom's Webster row builds its own `RunDeps` in `loomCLI.wire`, so that literal sets `ReadOnly` to `fabricengine.IsReadOnlyCommand` too, beside its `RefMatcher`; without it `runExitAuditCrossCheck` under the loom would keep every read-only reference a correctness halt.
`doc.go`'s audit section and the refusal-spec rows `audit: fabric-reference` and the webster section's severity preamble say a fabric reference with a read-only command is a policy guard that warns and records, and every other one a correctness halt.
Bound: the classifier sees static shape only, so a command behind `bash -c`, a variable or a substitution keeps its correctness class.
`TestClassifyViolation` gains a row per side: an accepted command is policy, a rejected one correctness, a nil classifier correctness; `TestRecordBatch_AuditOutcomes` gains the acceptance rows: a fork transcript with only read-only fabric references records done with audit warnings, and one with `lyx fabric add` fails as today.
Each pass-through is pinned where it is reached: `TestRecordBatch_ForgedTerminalRecordFails` gains a case whose unseen fork transcript holds only a read-only fabric reference, which `auditTerminalFork` leaves terminal with no failed record; `TestRun_RunExitRefusals` gains a row whose run-exit fork transcript holds only a read-only fabric reference, which warns and leaves the outcome done.
The CLI supply is pinned at the verb: `TestRecordBatchCmd_Envelope` gains a row whose fake fork audit's `BashCommands` hold only a read-only command naming the fabric repository, which records done with the audit warning on the envelope, so dropping `IsReadOnlyCommand` from `recordBatchCmd`'s deps fails it.
`TestWire_StandaloneMode` in webstercli's `wiring_test.go` asserts `runDeps().ReadOnly` is set, so `lyx webster run`'s supply is pinned; recover-batch's supply is pinned by card 16's `audit_not_acceptable` envelope row, which only an accepting classifier reaches.
`TestWire_DefaultConfig` asserts the loom's wired `runDeps.ReadOnly` is set, beside its `RefMatcher` assertion.

**ImpactSummary:** ClassifyViolation re-signed with its three callers in websterengine and its one test; three deps structs gain an optional field that webstercli fills in one place each and the loom's wiring fills in its own RunDeps literal.
