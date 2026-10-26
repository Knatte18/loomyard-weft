# Catch a renamed symbol a later card targets, at plan and batch end

A batch could rename or remove a symbol that a later card still names and record cleanly.
The next `begin-batch` then refused with `plan_conflict: tree_moved`, after the context that could have answered was gone.
In the case that prompted this, a card renamed `NewBeam` to `NewPawl` while its tests `TestNewBeam` and `TestBeamRouting` were listed under `Edit` only.
The implementer renamed the tests anyway, and quarry could offer the rename only as an evidence-tier candidate, which was an informational warning.

Three layers now catch it.

## Plan writer and plan check

The plan writer stencil asks a card that renames a symbol to declare, as `Rename` pairs on the same card, the renames of the tests named after it: `Test<Name>` and `Test<Name>_<case>`, or `Test<Type>_<Method>` for a method.

The new plan-format check `rename-leaves-test` (`internal/planparser`, row 53 of `loom-plan-spec.md`) enforces it under `language: go`.
A symbol `Rename` pair draws one blocking finding for each same-unit `Edit` ref naming a test after the symbol that no `Rename` pair of the plan renames.
The finding goes on the renaming card, where the missing pair belongs, and names the card that edits the test.
`TestNameX` is not `Name`'s test, since the base must be followed by nothing or `_`.

## Batch-end hand-back

`planindex.Delta` gains `RemovedTargets`, implemented in `internal/planglyph` as the read-only, blocking `later-target-removed` finding.
It reports a symbol the batch's delta deleted that a pending card targets or uses.
A symbol is exempt when quarry renamed it exactly, when the report lists it under `renames`, or when one of the batch's own cards declares it as a `Delete` target or a `Rename` old side.
Its finding names any evidence-tier rename candidates.

`postBatchChecks` runs it right after the shared delta and before handle binding, so on a refusal the only plan rewrite already made is the C# re-sign rebind, which a retried record-batch already re-ran before this change.
Only `RecordBatch` turns the check on, for an OK report.
The refusal is `*LaterTargetRemovedError` (`ErrLaterTargetRemoved`).
It leaves the batch open with its report in place and sets the new `BatchState.LaterTargetsReturned`.
The CLI saves the whole state, syncs fabric, and exits non-zero with `{"later_target_removed": true, "batch": "NN-<slug>"}`.

The Master stencil gains the ladder rung: send the refusal verbatim to the batch's fork by `SendMessage`, and call `record-batch` again on its completion notification.
If the fork cannot be reached, call `record-batch` again at once.
The implementer body gains "A batch handed back": restore the name in a commit or list the rename under `renames`, then rewrite the report with a fresh `head_sha`.
It also says that a test named after a renamed symbol is renamed only when the plan declares its pair.

## Design decisions

- The hand-back fires once per bracket.
  A second `record-batch` records the batch exactly as before, with DetectDrift's later-card warning and the later card's own `begin-batch` refusal, so a fork that cannot be reached, such as after a Master resume, never wedges the batch.
  The worst case is one extra fork turn per batch.
- recover-batch never hands back, since a recovery strand has ended by the time its report is read.
  A FAILED or NEEDS_PARENT report never hands back either.
- A declared removal that a later card still names stays out of the hand-back.
  It is a plan defect, owned by plan validation, `LaterDeleteReferences` and the existing warning.
- The board entry's "the batch gets a departure" is read as handing the batch back to its fork with the finding, not as a `Pattern-Departure:` trailer.
- `rename-leaves-test` runs under `go` only, since it reads Go's test naming convention, and it is listed with the other Go-only checks in the spec.

## Tests

- `TestValidate_RenameLeavesTest` covers exact and `_case` test names, a method test, a longer name, another unit, a declared test pair and a `csharp` plan.
- `TestRemovedTargets` covers later-card findings, candidate naming, a reported rename, a declared delete or rename, and an unreferenced removal.
- `TestRecordBatch_PostBatchChecks` now covers the hand-back for a plain removal and for an evidence-tier candidate, followed by the second call that records with the warning.
  Its fingerprint-restamp step starts past the hand-back.
- A recover-batch row checks that an unreported removal records the recovered batch with the warning.
- `TestRecordBatchCmd_LaterTargetRemovedEnvelope` checks the envelope and the saved `laterTargetsReturned` flag.
