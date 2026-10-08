# Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base

Both batches reported `status: done`.

## Batch 01 — pattern-fill-directive (cards 1–4), head `2ce070f1a`

- The directive fill is exported so a caller holding stencil and PATTERN bytes renders it as `Directive` does.
- `record-batch` resolves an abbreviated `head_sha` to the one commit it names and refuses an unresolvable one.
- `recover-batch` and `reset --to report-head` resolve `head_sha` the same way.
- A batch's own bracket transcripts that wrote the report count toward attribution and are re-audited.
- Deviation: `cmd/lyx/testdata/test-budget.yaml`, outside the declared file-ops, for the test-count budget of the new tests.

## Batch 05 — derived-overview-frame (cards 5–7), head `d4b4903fa`

- A state without an overview frame hash derives it from the stored plan baseline copy.
- `tokencount` measures each Merriam session's start at its prompt Read.
- `tokencount` reports a measured Merriam fixed context over the texts `MerriamBase` counts.
- Deviation: `internal/websterengine/merriambase.go`, outside the declared file-ops, for the `merriamFixedContext` comment change card 7 asks for.
