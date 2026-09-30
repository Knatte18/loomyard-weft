# Card 10 — post-batch-failures

**Edit:**
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/recordbatch_test.go`

**Uses:**
- `internal/websterengine/auditledger.go`
- `internal/websterengine/batchfail.go`
- `internal/websterengine/rebaseline.go`

**Intent:**
Finish D6 and D9 for the mechanical post-batch pass: a finding about the batch's own cards fails the batch, and a finding about a later card warns on this batch and refuses at that card's own begin-batch.

`postBatchChecks`:

- Done-check findings and blocking bind findings stay `ErrCardNotDone`-wrapped errors: both concern the batch's own cards.
- `DetectDrift`'s blocking findings (`plan-references-deleted-symbol`) no longer return an error.
  Each becomes a warning prefixed `later card:`: drift runs over the pending plan, which already excludes this batch's cards, so every such finding concerns a card still to be built (D9).
  D9 routes it through D5's recording path, so `postBatchInputs` gains `Batch *BatchState` and each finding is recorded with `recordBatchWarning` (card 2) under the identity `later-card-drift:<finding text>` and class `later-card-drift`.
  Its text joins the returned warnings only when newly added, so it reaches `state.json` once, the envelope, and card 15's `## Audit warnings` section through `RecordedAuditWarnings`.
  Both callers, `RecordBatch` and `PersistRecoveryTerminal`, pass the record they are finishing.
  Update the doc comment's severity-split paragraph.

`RecordBatch`: an `ErrCardNotDone` from `postBatchChecks` goes through `failBatch` (reasons = the findings; no suspect paths; the report archived; transcripts already consumed), returning `RecordResult{Failed: true, …}` with the `*BatchFailedError` and the pass's warnings.
Any other `postBatchChecks` error is still returned as-is with nothing marked terminal.
`ErrCardNotDone`'s doc comment says record-batch now converts it into a failed batch.

`ErrPlanDrifted`'s wrapping message in `BeginBatch` ends `way forward: edit the plan so the named cards match the tree, run "lyx webster rebaseline", then begin-batch NN again`.

Tests (integration), each from a real repo:

- An OK report whose Create target does not resolve → batch terminal `failed`, report archived, `errors.Is(err, ErrBatchFailed)`, `ErrCardNotDone`'s findings in the digest reasons; `RecoverSpawnOrAttach` is exercised in card 11.
  Rewrite `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` to this.
- Batch 1 deletes a symbol card 2 still references → batch 1 records `done` with a `later card:` warning, also present once in batch 1's `AuditWarnings` in state.json; `BeginBatch(2)` refuses with `ErrPlanDrifted` naming `lyx webster rebaseline`; after the test edits card 2 to drop the reference and calls `Rebaseline`, `BeginBatch(2)` proceeds.

**ImpactSummary:** record-batch fails its own batch instead of leaving it non-terminal, and drift about later cards becomes a warning recorded once in the batch's state; `postBatchInputs` gains one field that record-batch and recover-batch, its only two callers, fill; recover-batch's not-done outcome is aligned in card 11.
