# Card 4 — a batch's own bracket transcripts count toward attribution and are re-audited

**Create:**
- `plan:internal/websterengine#isOwnReportWrite` -> `func isOwnReportWrite(workdir, write, ownReport string) bool`

**Edit:**
- `internal/websterengine#BatchState`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#CheckFork`
- `internal/websterengine#ClassifyAttribution`
- `internal/websterengine#ErrNoForkTranscripts`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D2 (#457): a fork stopped and resumed across a no-report record-batch appends to a transcript that call already attributed, so the next call saw zero new transcripts and archived a good report.
The bracket's own transcripts now count toward attribution when they wrote the report, and every one of them is re-audited in full.

State:
- `BatchState` gains the field `BracketTranscripts []string` (JSON `bracketTranscripts`, omitempty): the transcripts record-batch attributed to this batch during its current bracket.
  begin-batch and recover-batch build a fresh `BatchState` and do not carry it, so every begin-batch, a re-begin of a non-terminal batch included, and every recovery spawn opens an empty list; `ForkTranscripts` keeps carrying over as today.
  The field comment says so.

`RecordBatch`:
- The seen set handed to `AuditForksIncremental` is `SeenForkTranscripts` minus `bs.BracketTranscripts`, so the engine re-reads the bracket's own transcripts in full.
- `SettleRetry` is called as today with the full `SeenForkTranscripts`: it waits out its window for a transcript new since the run's boundary.
- From the settled audit, the bracket's own transcripts are the forks whose `TranscriptPath` is in `bs.BracketTranscripts`.
  One of them counts toward attribution only when one of its `WritePaths` is the batch's own report path, judged by `isOwnReportWrite`.
- `ClassifyAttribution` gets the distinct union of the new transcripts and the counted own ones: zero is `ErrNoForkTranscripts` and the report is archived as today, one attributes, and more than one is the multi-transcript warning.
- `CheckFork` and `ForkWarnings` run over the new transcripts and every bracket transcript the audit returned, counted or not, so a resumed fork's later writes are audited; a finding already dispositioned is dropped by its identity as today.
- `SeenForkTranscripts`, `bs.ForkTranscripts` and `bs.BracketTranscripts` each advance by the new transcripts only, with no duplicate entries, on every call that reaches the advance, the no-report call included, so no other batch's record-batch and no `auditTerminalFork` can claim a transcript this bracket holds.

`isOwnReportWrite` in `audit.go` holds the comparison `CheckFork` already makes to exempt the fork's own report: the write path resolved against workdir (`resolveWritePath`) equals the cleaned report path, or both canonicalize to the same path.
`CheckFork` calls it in place of its inline comparison, with no change to what it reports.

What this lets through, and the bound: a carried-over transcript lets through only a report its own fork wrote with a tool the audit sees (Write, Edit or NotebookEdit).
A report Master wrote, a report no bracket transcript shows a write to, and a bracket whose only transcripts belong to an earlier bracket of the same batch number or to another batch all still meet `ErrNoForkTranscripts`.
The cost is at most one settle window per call on a bracket that already attributed a transcript.

Finding identity across re-reads (the record's open risk): `forkKey` keys a fork finding by transcript, class and per-class ordinal in transcript order, and a transcript only grows, so a re-read finds each earlier finding under the same key and `isDispositioned` drops it; the tests below pin this.

Wording:
- `ErrNoForkTranscripts`' text says there is no counted fork transcript in the batch's current bracket: none new since the previous batch boundary, and none of the bracket's own that wrote the batch's report.
  It keeps the machine-local caveat, which `TestRecordBatch_ZeroNewTranscriptsArchivesReport` asserts.
- `ClassifyAttribution`'s doc comment speaks of the counted transcripts rather than the new ones, and its multi-transcript warning says that many fork transcripts count toward the batch.
  Grep the tests for the old warning and error texts and update every assertion on them in this card.

Tests:
- `recordbatch_test.go`: one table-driven test over a fork resumed across a no-report call, with the scripted audit recording the seen set it is handed, and rows for:
  a first call with no report (the transcript attributed), then a second call after the same transcript grew with a write to the report and the report landed, which records the batch instead of archiving, with the seen set handed to the engine excluding the bracket's transcript;
  on that second call, a policy finding from the transcript's later part reported, and the finding the first call dispositioned not reported again;
  after a no-report call, a report that no bracket transcript shows a write to archived as unattributable with `ErrNoForkTranscripts`;
  after a no-report call, a fork re-launched in the open bracket whose transcript appears only after the first fetch attributed to this batch within the settle window, and not counted by the next batch's record-batch;
  a no-report call's transcript not claimed by another batch's record-batch nor by `auditTerminalFork`.
- `beginbatch_test.go`, in `TestBeginBatch_Record`: a re-begin of a non-terminal batch whose record holds bracket transcripts writes a record with none, while `ForkTranscripts` carries over; and `recordbatch_test.go` pins that zero new transcripts after that re-begin is still `ErrNoForkTranscripts`.

Docs: `doc.go`'s attribution section states that a bracket's own transcripts that wrote the report count toward attribution, that all of them are re-audited, and that begin-batch and recover-batch open an empty bracket list.
`refusal-spec.md`'s `report unattributable` row is reworded to the new trigger: no begin record, no counted fork transcript in the batch's current bracket (none new since the previous batch boundary and none of the bracket's own that wrote the report), or no transcript directory.

**ImpactSummary:** `state.json` gains one omitempty per-batch field; record-batch's attribution and audit set widen to the bracket's own transcripts; `CheckFork`'s findings are unchanged.

**Commit:** 4: webster record-batch: a batch's own bracket transcripts count toward attribution and are re-audited (#457)
