# Card 1 — plan-conflict-class

**Create:**
- `plan:internal/websterengine#ErrPlanConflict` -> `var ErrPlanConflict error`
- `plan:internal/websterengine#PlanConflictError` -> `type PlanConflictError struct`
- `plan:internal/websterengine#PlanConflictError.Error` -> `func (e *PlanConflictError) Error() string`
- `plan:internal/websterengine#PlanConflictError.Unwrap` -> `func (e *PlanConflictError) Unwrap() error`
- `plan:internal/websterengine#ConflictCausePlanDefect` -> `const ConflictCausePlanDefect = "plan_defect"`
- `plan:internal/websterengine#ConflictCauseTreeMoved` -> `const ConflictCauseTreeMoved = "tree_moved"`
- `plan:internal/webstercli#TestBeginBatchCmd_PlanConflict` -> `func TestBeginBatchCmd_PlanConflict(t *testing.T)`
- `plan:internal/webstercli#TestFingerprintRefusals_CarryPlanDrifted` -> `func TestFingerprintRefusals_CarryPlanDrifted(t *testing.T)`

**Delete:**
- `internal/websterengine#ErrPlanDrifted`

**Edit:**
- `internal/websterengine#BeginBatch`
- `internal/websterengine#BeginResult`
- `internal/websterengine_test#TestBeginBatch_ReResolvesPlanAtDispatch`
- `internal/websterengine_test#TestBeginBatch_AlreadyBuiltCardsAreNotReResolved`
- `internal/websterengine_test#TestBeginBatch_RestampsFingerprintEvenWhenPlanDrifts`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `contracts/stencils#websterPlanDriftSection`
- `contracts/stencils#wordingClaims`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/doc.go`

**Intent:**
Decision D1, D4 and D5's webster row: begin-batch's dispatch refusal stops calling itself plan drift and carries its real cause.

In `internal/websterengine/beginbatch.go`, `ErrPlanDrifted` is deleted (begin-batch is its only raiser; record-batch and recover-batch map `ErrFingerprintMismatch` to `plan_drifted` and are untouched).
It is replaced by the sentinel `ErrPlanConflict` (`errors.New("webster: plan re-resolution at begin-batch reported a blocking finding")`) and the error type `PlanConflictError`, with fields `Batch int`, `Cause string` and `Findings []string` (every blocking finding as `Finding.Error` renders it, in the order `ValidateDispatch` reported them).
`Unwrap` returns `ErrPlanConflict`, and `Error` renders the existing message with the class added: `webster: plan re-resolution at begin-batch reported a blocking finding (plan_conflict: <cause>): <findings joined by "; ">; way forward: edit the plan so the named cards match the tree, run "lyx webster rebaseline --card NN" naming each card you edited, then begin-batch NN again`, NN the zero-padded batch.
The two cause constants are the only values `Cause` takes.

`BeginBatch` returns `&PlanConflictError{...}` at the blocking-findings return, with `Cause` set to `ConflictCausePlanDefect` when the `begun` slice `DispatchScope` returned is empty and `ConflictCauseTreeMoved` otherwise.
`begun` is computed before this call records its batch, so a first begin of batch 1 is `plan_defect` and a re-begin of a recorded batch is `tree_moved`;
the comment at the return says so.
The fingerprint re-stamp ahead of it, the quarry-unavailable path (`resolveErr` returned bare, never a `PlanConflictError`) and every later step are unchanged.
`BeginResult`'s `Advisories` comment names `ErrPlanConflict` in place of `ErrPlanDrifted`.

In `internal/webstercli/beginbatch.go`, `beginBatchCmd`'s refusal arm replaces the `ErrPlanDrifted` branch with `errors.As(err, &conflict)` for a `*websterengine.PlanConflictError`, printing `output.ErrFields(out, err.Error(), map[string]any{"plan_conflict": conflict.Cause})`;
the envelope never carries `plan_drifted`.
The command's `Long` text names the `plan_conflict` refusal and its two causes.
The webster refusal note (`noteRefusals`) records the class unchanged, since it copies every envelope key into the note's fields.

The master stencil's section headed by `websterPlanDriftSection` is retitled "## A plan-drift or plan-conflict refusal ends your run as stuck — do not retry the verb", and the constant follows the new heading's prefix.
The section says: a `begin-batch` refusal carrying `{"plan_conflict": "<cause>"}` (`plan_defect` or `tree_moved`) means begin-batch's re-resolution found the plan contradicts the tree before the batch would have been built;
the master writes `outcome: stuck` with a `stuck_reason` quoting the refusal verbatim and stops, without retrying the verb or trying another batch;
under loom the stuck run is routed to plan review, and a standalone run's way forward is the refusal's own.
The `record-batch`/`recover-batch` `{"plan_drifted": true}` sentences and their `lyx webster restore-plan` way forward stay.
`wordingClaims`' webster-master entries change accordingly: the begin-batch claim now requires `"plan_conflict"` and "do not retry the verb" inside the section, a new claim requires the section to say a `plan_conflict` stuck is routed to plan review under loom and that a standalone run's way forward is the refusal's own, and the record-batch/recover-batch `plan_drifted` claim stays.

`contracts/specs/refusal-spec.md`'s webster section replaces the `plan drifted` row with a `plan conflict` row: trigger "begin-batch's re-resolution of the plan against the tree finds a blocking defect; the refusal carries `plan_conflict` with cause `plan_defect` when no batch is begun and `tree_moved` when one is", class correctness halt, the same way forward.
`internal/websterengine/doc.go`'s dispatch-scope section gains one sentence naming the refusal's class and how the begun set decides its cause.

Tests: `TestBeginBatch_ReResolvesPlanAtDispatch`'s blocking row asserts `errors.As` yields a `PlanConflictError` with `Batch` 1, cause `plan_defect` and the bare-symbol finding in `Findings`, and that the message keeps `lyx webster rebaseline --card NN`;
a new row puts the bare-symbol `Uses` on the fixture's second card, records batch 1 as begun and not terminal, begins batch 2 and asserts cause `tree_moved`;
the quarry-unavailable row asserts the error is not an `ErrPlanConflict`.
`TestBeginBatch_AlreadyBuiltCardsAreNotReResolved` and `TestBeginBatch_RestampsFingerprintEvenWhenPlanDrifts` assert `ErrPlanConflict` in place of `ErrPlanDrifted`.
The new integration test `TestBeginBatchCmd_PlanConflict` in `internal/webstercli/verbs_test.go` (package budget has room) is the table every begin-batch conflict and conflict-record row of this plan joins.
Its first row runs the fixture's single Create card, whose target file is committed into the worktree and which also `Uses` the bare `sub.Foo`, so it carries two blocking findings (create-already-exists and bare-symbol-target) with no batch begun.
It drives `beginBatchCmd` wrapped by `noteRefusals`, over `newVerbsFixture` with `frictionDir` set to a temp dir, and asserts exit 1 and `"plan_conflict":"plan_defect"`, a message naming `lyx webster rebaseline --card NN` and no `plan_drifted`, and a refusal note in the friction dir holding `plan_conflict: plan_defect`.
The `tree_moved` cause is a pure engine decision the CLI only copies into the envelope, so the engine row above is its coverage and the CLI table carries no second cause row.
The new integration test `TestFingerprintRefusals_CarryPlanDrifted` in the same file pins acceptance criterion 4, which no test drives today: a table of `record-batch 1` over a batch 1 recorded begun and not terminal, and `recover-batch 1` over a batch 1 recorded terminal failed, each after the fixture's card file is edited past the recorded fingerprint, asserts exit 1 and `"plan_drifted":true`.
`TestStencilClaims` reaches the claims; `TestMasterStencilCoversEverySingleFlagRefusal` keeps passing, since `plan_conflict` carries a string, outside its single-flag shape, and `plan_drifted` stays in record-batch and recover-batch.

**ImpactSummary:** One sentinel deleted with one production caller (`beginBatchCmd`) and three engine tests retargeted in the same card; the envelope class seen by Merriam and the refusal note changes from `plan_drifted` to `plan_conflict` for begin-batch only.
