# Card 5 — archive-unrecorded-report

**Edit:**
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/beginbatch.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/beginbatch_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Implement decision D5 (#402): `begin-batch` handles a report left with no begin-batch record itself, instead of bouncing begin-batch → record-batch → begin-batch.

In `BeginBatch`'s pre-existing-report step: when the report exists and `State.Batches` has no record for the batch number, archive it through `archiveStaleReport` (the timestamped rename `record-batch`'s unattributable path uses) and proceed; return the archived path in a new `BeginResult.ArchivedReport` field, and have `internal/webstercli/beginbatch.go` put it in the envelope as `archived_report` (omitted when empty) and include it in the fabric sync, as record-batch does for its archive.
When state.json does record the batch, the refusal stays, and its message now states the record it saw beside the existing way forward: "begun and not terminal" or "terminal with status <status>".
Run the archive after the dispatch validation and the batch lookup, at the point the refusal sits today, so an earlier refusal never moves a file.

Bound, stated in the code comment: only when no record for that batch number exists; a recorded batch's report is never archived by begin-batch.

Docs: update the refusal-spec row "report already present" to say it fires for a batch state.json records and that the message names the record's state; update `doc.go` where it describes begin-batch's report refusal; update the `begin-batch` `Long` text in `internal/webstercli/beginbatch.go` to say a report with no begin-batch record is archived.

Tests:
in `beginbatch_test.go`, extend `TestBeginBatch_PreExistingReportRefused` (or its table) with the no-record case: the report is renamed to an archive path under the reports dir, the call succeeds, `ArchivedReport` names it and the batch is recorded; and with a recorded non-terminal batch and a recorded terminal one the call refuses, names the state it saw, and moves nothing.
`TestBeginBatch_WayForward_ReportExistsIsRecorded` seeds a report on a fixture with no batch record, which D5 now archives instead of refusing, so it is edited to seed a begun, non-terminal batch record beside the report, which keeps pinning the `record-batch` way forward and the report left in place.
In `internal/webstercli/verbs_test.go`, drive `begin-batch` through the cobra command on the verbs fixture with a report on disk and no record: exit 0, the envelope carries `archived_report`; with a record present it exits 1 naming the record's state.

**ImpactSummary:** One branch in BeginBatch's report check plus an archived-path field on its result and envelope; the recorded-batch refusal keeps its way forward with a richer message.
