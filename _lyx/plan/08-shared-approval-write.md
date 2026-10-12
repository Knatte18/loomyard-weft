# Card 8 — shared-approval-write

**Create:**
- `plan:internal/landingshed#RecordApproval` -> `func RecordApproval(approvalPath, rejectionPath string, a Approval) error`
- `plan:internal/landingshed#RecordRejection` -> `func RecordRejection(approvalPath, rejectionPath string, r Rejection) error`

**Edit:**
- `internal/landingshed#TestApproval_RoundTripAndOverwrite`
- `internal/landingshed#TestRejection_RoundTripAndOverwrite`
- `internal/loomcli#approveDeps`
- `internal/loomcli#approveVerb`
- `internal/loomcli#loomCLI.approveCmd`
- `internal/loomcli#approveFixture`
- `internal/loomcli#TestApproveVerb_Refusals`
- `internal/loomcli#TestApproveVerb_Success`
- `internal/loomcli#TestApproveVerb_RemoveRejectionFailureWritesNoApproval`
- `internal/loomcli#rejectDeps`
- `internal/loomcli#rejectVerb`
- `internal/loomcli#loomCLI.rejectCmd`
- `internal/loomcli#rejectCalls`
- `internal/loomcli#rejectFixture`
- `internal/loomcli#TestRejectVerb_Success`

**Intent:**
Make the approval and rejection write one shared path, so card 9's `PR-Align` applies an agent verdict through the same code `lyx loom approve` and `lyx loom reject` run, never a copy (D5).

`RecordApproval` removes the rejection record at `rejectionPath` through `RemoveRecord`, then writes `a` through `WriteApproval`, so the latest decision wins.
A failed removal writes no approval and returns the removal's error.
`RecordRejection` is its mirror: it removes the approval, then writes `r` through `WriteRejection`.
Both live in `approval.go` and `rejection.go` beside the record they write.

The verbs' two record closures collapse into one each:
- `approveDeps.removeRejection` and `writeApproval` become `recordApproval func(landingshed.Approval) error`. `approveVerb` calls it once, keeping its error envelope, and `approveCmd` wires it to `landingshed.RecordApproval(loomengine.LoomApprovalPath(location), loomengine.LoomRejectionPath(location), a)`.
- `rejectDeps.removeApproval` and `writeRejection` become `recordRejection`, wired to `landingshed.RecordRejection` the same way.

The order guarantee moves into `landingshed`, so the verb tests stop asserting it:
- `TestApproval_RoundTripAndOverwrite` gains subtests: `RecordApproval` removes a pending rejection and writes the approval, and a rejection path that cannot be removed (a non-empty directory) leaves no approval file.
- `TestRejection_RoundTripAndOverwrite` gets the mirror subtests for `RecordRejection`.
- `approveFixture` fakes `recordApproval`, recording the written approvals. `TestApproveVerb_Refusals` swaps `recordApproval` where it swapped `writeApproval`. `TestApproveVerb_Success` drops its order assertion and its `//testtiming:keep` reason's order clause.
- `TestApproveVerb_RemoveRejectionFailureWritesNoApproval` makes `recordApproval` fail as the shared path does when the removal fails ("disk full", nothing recorded), and asserts one error envelope naming it. Its doc line says the record step's removal failing writes no approval.
- `rejectCalls` drops `order`, and `rejectFixture` fakes `recordRejection`. `TestRejectVerb_Success` drops its `remove,write` order assertion, its doc comment says the rejection is recorded through the shared path, and it keeps the written record and envelope checks.

**ImpactSummary:** Two record closures per verb become one, both verbs and their tests move together, and the records' on-disk format and the verbs' envelopes are unchanged.
