# Card 12 — done-checks-name-the-batch

**Edit:**
- `internal/websterengine#failCardNotDone`
- `internal/websterengine/recordbatch_test.go`
- `//contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#postBatchChecks`

**Intent:** Make a done-check refusal read as the batch's own (D6).
`failCardNotDone`'s reasons open with the batch's id and its card list, in the form `batch 01-headcontains-bounded-walk holds cards 1-…, 2-…, 3-proc-group-kill`, built from `cardNotDoneInputs.Cards`, so a finding keyed on a later card of the batch is seen to belong to it.
The way forward is unchanged.
`TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` gains a row over a three-card batch beside unbegun cards that pins two things: the fake index's `DoneChecks` receives the batch's three cards only, which `postBatchChecks` already hands it, and the failure reasons open with the batch id and its card list.
The refusal-spec row `card not done` says the reasons open with the batch and its cards.

**ImpactSummary:** One unexported helper's text; record-batch and recover-batch both reach it and their envelopes change only in the reason's opening.
