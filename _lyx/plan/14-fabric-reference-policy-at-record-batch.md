# Card 14 — fabric-reference-policy-at-record-batch

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
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.runDeps`
- `internal/webstercli#websterCLI.recordBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `//contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#IsReadOnlyCommand`

**Intent:** Class a fabric reference whose command is read-only as policy, so the batch records an audit warning and passes (D8).
`ClassifyViolation` takes the read-only classifier by injection, as `AcceptBatchFabricReference` already does, so websterengine keeps importing no `fabricengine`: a `fabric-reference` whose `Command` the classifier accepts is policy, every other fabric reference stays correctness, and a nil classifier accepts nothing.
`RecordDeps`, `RunDeps` and `RecoverDeps` gain `ReadOnly func(cmd string) bool`, and `RecordBatch`, `auditTerminalFork` and `runExitAuditCrossCheck` pass theirs through.
`webstercli` supplies `fabricengine.IsReadOnlyCommand` on all three: `runDeps`, record-batch's deps and recover-batch's deps.
`doc.go`'s audit section and the refusal-spec rows `audit: fabric-reference` and the webster section's severity preamble say a fabric reference with a read-only command is a policy guard that warns and records, and every other one a correctness halt.
Bound: the classifier sees static shape only, so a command behind `bash -c`, a variable or a substitution keeps its correctness class.
`TestClassifyViolation` gains a row per side: an accepted command is policy, a rejected one correctness, a nil classifier correctness; `TestRecordBatch_AuditOutcomes` gains the acceptance rows: a fork transcript with only read-only fabric references records done with audit warnings, and one with `lyx fabric add` fails as today.

**ImpactSummary:** ClassifyViolation re-signed with its three callers in websterengine and its one test; three deps structs gain an optional field the CLI fills in one place each.
