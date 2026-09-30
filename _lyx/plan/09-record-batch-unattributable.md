# Card 9 — record-batch-unattributable

**Edit:**
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/audit.go`
- `internal/websterengine/recordbatch_test.go`

**Uses:**
- `internal/websterengine/beginbatch.go`

**Intent:**
Implement D7: a report record-batch cannot attribute is archived and the batch re-driven through `begin-batch`, which breaks the begin-batch/record-batch/recover-batch refusal ring.

In `recordbatch.go`:

- Add `ErrReportArchived` and `ReportArchivedError` (`Number`, `Batch`, `ArchivedTo`, `Cause error`), whose message states why the report could not be attributed, where it was archived, and ends `way forward: lyx webster begin-batch NN re-drives the batch`.
  `Unwrap` returns both `ErrReportArchived` and `Cause` (a `[]error` unwrap), so `errors.Is` still matches `ErrNoBeginRecord` or `ErrNoForkTranscripts`.
  `RecordResult` gains `ArchivedReport string` for the caller's fabric sync.
- No record for the batch (`!ok || bs == nil`): resolve the slug through `findBatch`; archive the report through `archiveStaleReport` when one exists and return `ReportArchivedError` with `Cause: ErrNoBeginRecord`;
  with no report, return `ErrNoBeginRecord` wrapped in a message naming `lyx webster begin-batch NN`.
- A terminal record: refuse without mutating anything, naming the recorded status and the way forward — `done`: continue with the next batch; `failed`, `stuck` or `dead`: `lyx webster recover-batch NN`.
  It no longer shares `ErrNoBeginRecord`'s wording.
- Zero new fork transcripts after the settle window, or no transcript directory for the bracket's session on this machine (`fs.ErrNotExist`): archive the report when present, leave the batch record begun and non-terminal, and return `ReportArchivedError` with the matching cause.
  With no report either, return the cause wrapped with the same begin-batch way forward.

In `audit.go`, `ErrNoForkTranscripts`'s text drops the "moving the batch's report file out of the reports dir" recourse and keeps the diagnosis; its doc comment describes D7's archive-and-re-drive instead of the three-verb circle.

Tests (integration):

- A report present with no begin record → `errors.Is(err, ErrReportArchived)`, the report moved to its archive name, message contains `lyx webster begin-batch 02`; `BeginBatch(2)` then succeeds (its pre-existing-report refusal no longer fires).
- A begun batch, a report present, zero fork transcripts after the settle window → report archived, record still begun; `BeginBatch` re-begins it keeping its StartSHA.
- A missing session transcript directory → same outcome.
- A terminal `done` batch → refusal naming "next batch", report left in place.
- Rewrite `TestRecordBatch_NoBeginRecord`, `TestRecordBatch_ZeroNewTranscriptsHardErrorsEvenWithReport` and `TestRecordBatch_MissingSessionTranscriptNamesRecourse` to the new behavior.

**ImpactSummary:** Changes record-batch's three attribution refusals from terminal wedges to archive-and-re-drive; callers are webstercli's record-batch verb (card 12) and these tests.
