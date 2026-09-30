# Card 7 — batch-fail-primitive

**Edit:**
- `internal/websterengine/digest.go`
- `internal/websterengine/state.go`

**Create:**
- `internal/websterengine/batchfail.go`
- `internal/websterengine/batchfail_test.go`

**Uses:**
- `internal/websterengine/recoverbatch.go`

**Intent:**
Add the single primitive D6 uses to take a batch terminal-failed instead of leaving it wedged, so record-batch (cards 8–10) and recover-batch (card 11) share one implementation.

`digest.go` gains `DigestStatusFailed = "failed"` (the doc comment now lists five legal values and says `failed` is a batch whose report webster rejected on its merits) and `Digest.Reasons []string` (`reasons,omitempty`), set only on a failed digest.
In `state.go`, the doc comments of `BatchState.Terminal` and `BatchState.Status`, which list the terminal statuses, add `failed`.

`batchfail.go` declares:

- `ErrBatchFailed`, a sentinel.
- `BatchFailedError` (`Number int`, `Batch string`, `Reasons []string`, `SuspectPaths []string`, `ArchivedReport string`), whose `Error()` reads `webster: batch NN-<slug> failed: <reasons joined "; ">`, then `suspect paths: <paths>` when any, then `report archived to <path>` when archived, and ends `way forward: lyx webster recover-batch NN` (the recovery strand reads the digest and reverts or re-derives each suspect path).
  `Unwrap` returns `ErrBatchFailed`.
- `failBatch(in failBatchInput) (*BatchFailedError, error)`, where `failBatchInput` carries `State`, `Batch *BatchState`, `Number`, `Slug`, `ReportsDir`, `HeadSHA`, `Reasons`, `SuspectPaths`, `NewTranscripts` and `Now func() time.Time`.
  It appends `NewTranscripts` to `State.SeenForkTranscripts` and the batch's `ForkTranscripts` (skipping entries already present), archives the report through `archiveStaleReport` (an absent report archives nothing), sets the batch `Terminal`, `Status` `failed` and a `Digest` carrying `Batch`, `Status`, `HeadSHA` and `Reasons` (each suspect path is named in a reason line), and clears `State.CurrentBatch`.
  The returned error is only the archive's I/O failure, in which case nothing in `State` has been mutated yet: the archive runs first.

`batchfail_test.go` (untagged, `package websterengine`, files under `t.TempDir()`) covers: an OK report present → archived under the timestamped name and the live path is free; no report → nothing archived; the record is terminal `failed` with the reasons; transcripts are appended once even when passed twice; the error text contains `lyx webster recover-batch 03` and every suspect path; and `errors.Is(err, ErrBatchFailed)`.

**ImpactSummary:** Adds one Digest status value and one omitempty field; no reader validates the status set, and every reader treats `failed` as it treats `stuck` — terminal for `completedCards`, not done for `verifyEveryBatchDone`, printed verbatim by `lyx webster status` and the Master progress trail.
