# Card 7 — journal-and-report-paths

**Create:**
- `plan:internal/weaverengine#AttemptReportPath` -> `func AttemptReportPath(reportsDir string, number int, slug string, attempt int) string`
- `plan:internal/weaverengine#FixerReplyPath` -> `func FixerReplyPath(reportsDir string, fixer int) string`
- `plan:internal/weaverengine#JournalHeading` -> `func JournalHeading(batch string, attempt int) string`
- `plan:internal/weaverengine#AppendJournal` -> `func AppendJournal(path, heading, text string) error`
- `plan:internal/weaverengine#TestAppendJournal` -> `func TestAppendJournal(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#ReportFileName`
- `internal/runplan#Report`

**Intent:** Fix each strand's identity on disk and keep the run journal (D8, D10, D12).

- `AttemptReportPath` is `<reportsDir>/NN-<slug>-a<attempt>.yaml`, unique to the batch and attempt number, so every attempt's strand has its own output file and `ProbeGated` finds its run by that file alone.
- `FixerReplyPath` is `<reportsDir>/fix-<n>.md`, the verify fixer's only output file.
- `JournalHeading` renders `## Batch <NN-slug>, attempt <n>`.
- `AppendJournal` appends the heading and the text to `journal.md`, creating the file, and does nothing when the file already holds the heading line, so recording a report again after a crash adds no second entry.
  An empty text appends the heading alone: a report with an empty `journal` is accepted.
  It writes through a temp file and a rename (`runplan.WriteFileAtomic`), so a crash never leaves half an entry.

`TestAppendJournal` is tier0: two entries append in order; a repeated heading adds nothing; an empty text appends the heading; the paths' file names.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
