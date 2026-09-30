# Card 12 — webstercli-failure-envelopes

**Edit:**
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`

**Uses:**
- `internal/websterengine/batchfail.go`
- `internal/websterengine/recordbatch.go`

**Intent:**
Persist and commit what D6 and D7 changed before reporting the refusal, so the failed batch or the archived report is durable and the next verb sees it.

`record-batch` verb:

- `errors.Is(err, websterengine.ErrBatchFailed)`: save the whole mutated state (not the fingerprint-only re-baseline, which exists to drop mutations on other errors), release the lease, fabric-sync with the label `record-batch NN failed`, then emit `output.ErrFields` with `batch_failed: true`, `batch` and `warnings`.
- `errors.Is(err, websterengine.ErrReportArchived)`: nothing in state changed, so no save; fabric-sync with the label `record-batch NN report-archived` so the archive move is committed, then emit `output.ErrFields` with `report_archived: true` and `batch`.
- The `card_not_done` key is gone: record-batch no longer returns `ErrCardNotDone` to its caller.
- A fabric sync failure on either path is appended to the message, as on the success path.
- Every other error keeps today's fingerprint-only persistence.
- Update the file header and the command's `Long` to describe both envelopes.

`recover-batch` verb: a `*BatchFailedError` from `PersistRecoveryTerminal` saves the reloaded state, fabric-syncs `NN failed`, and emits `batch_failed: true` the same way.

Tests:

- `verbs_test.go` (integration): a record-batch over a fork-contract-write finding exits non-zero with `batch_failed: true` and a message containing `lyx webster recover-batch`, and state.json on disk shows the batch terminal `failed`; a record-batch with no begin record and a report present exits with `report_archived: true`, and the report is gone from the live path.
- `cli_test.go`: the envelope-key comment and test naming `card_not_done` now name `batch_failed` and `report_archived`.

**ImpactSummary:** Changes record-batch/recover-batch's error envelopes and persistence on two sentinels only; Master's stencil (card 13) is their only reader.
