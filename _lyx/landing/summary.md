# Catch a renamed symbol a later card targets before begin-batch refuses

A batch that renames or removes a symbol a later card still targets used to surface only at the next `begin-batch`, as `plan_conflict: tree_moved`.
Seen in a past run: a card renamed `NewBeam` and `BeamRouting` as Rename pairs but left `TestNewBeam` and `TestBeamRouting` under `Edit`.
The implementer renamed the tests too and reported no departures, and the next batch's `begin-batch` refused because a later card still named the old tests.
Three layers now catch it, from plan writing to batch end.

## What changes

- **Plan writer.**
  The stencil `contracts/stencils/loom/loom-template-plan.md` tells a card that renames a symbol to declare, as Rename pairs on the same card, the renames of the tests named after it.
- **Plan validation.**
  A new blocking check `rename-leaves-test` in `internal/planparser` flags a Rename pair whose symbol has same-named tests left under `Edit`.
  It is attributed to the renaming card, names the card that edits the test, and its way forward is to add the test's rename as a Rename pair.
  It is documented in `contracts/specs/loom-plan-spec.md` and the package doc.
- **Batch end.**
  `lyx webster record-batch` refuses an OK fork report whose batch removed a symbol a pending card targets or uses, with `ErrLaterTargetRemoved`.
  The batch stays non-terminal, its report stays in place and its bracket stays open.
  The CLI saves the run state, syncs fabric and exits non-zero with `{"later_target_removed": true, "batch": "NN-<slug>"}`.
  The Master stencil gains a failure-ladder rung that sends the refusal to the batch's fork by `SendMessage` and calls `record-batch` again on the fork's completion.
  The implementer body says how a fork answers: restore the name, or list the rename under `renames`, commit any restore and rewrite its report with the new head.
  It also says a renamed test a card names is a rename the fork reports.

## Design decisions

- **`rename-leaves-test` is Go-only.**
  A Rename pair whose old side is `unit#Name` names the test base `Name`, and `unit#T.M` names `T_M`.
  An `Edit` ref to `Test<base>` or `Test<base>_<suffix>` that no Rename pair renames is a finding; `TestNewBeamer` is not in `NewBeam`'s set.
  The unit match accepts the external test package (`<dir>_test`) as well as the symbol's own unit.
  The check is silent under `csharp` and `none`, whose test naming differs.
- **The batch-end check is read-only and runs before handle binding.**
  `planindex.Delta` gains `RemovedTargets`, implemented in `internal/planglyph` as the blocking finding `later-target-removed`, one per card, naming any evidence-tier rename candidates quarry offered.
  It runs right after the shared delta in `postBatchChecks` and before `BindHandles`, so a refusal leaves the plan unrewritten and a retried record-batch sees no half-applied bind.
  A symbol draws no finding when quarry renamed it exactly, when the report lists it under `renames`, or when one of the batch's own cards declares it as a Delete target or a Rename pair's old side.
- **The hand-back happens once per batch.**
  A new additive field on `BatchState` records that the batch was returned, read tolerantly.
  A second `record-batch` on a returned batch records it as today, with the later-card warning and the later card's own `begin-batch` refusal, so a fork that cannot be reached cannot block the batch.
  It never fires on a FAILED or NEEDS_PARENT report, and `recover-batch` never refuses on it.
- "A later card targets" includes `Uses:` refs, matching the reference index drift detection already reads.
- "The batch gets a departure" is read as handing the batch back to its fork with the finding, not as a `Pattern-Departure:` trailer.
- Out of scope: `recover-batch`, `begin-batch`'s own refusal and the plan-review rubric.

Docs updated: `internal/websterengine/doc.go`, `internal/planglyph/doc.go`, `internal/planparser/doc.go` and the `record-batch` help text.

## Check by hand

- Nothing was run end to end against a real plan and fork; the layers are covered by unit and engine tests only.
  Run a plan whose card renames a symbol with same-named tests and watch the plan check and the hand-back.
- `internal/planparser/validate.go` `checkRenameLeavesTest`: confirm a test in the external test package (`foo_test#TestX` beside a renamed `foo#X`) is flagged.
- `internal/websterengine/recordbatch_git_test.go` `TestRecordBatch_PostBatchChecks`: confirm a FAILED report is never handed back over a removed later target.
- `internal/websterengine/recordbatch_git_test.go` near the mixed rename/delete subtest: confirm an exact quarry rename draws no hand-back.
- `internal/planparser/validate_test.go` `TestValidate_RenameLeavesTest`: confirm a plan with language `none` draws no finding.
- Quarry may report a symbol deleted whose file moved between packages; the later card then sees the refusal once and the existing warning after.
