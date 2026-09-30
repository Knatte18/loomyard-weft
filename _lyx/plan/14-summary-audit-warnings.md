# Card 14 — summary-audit-warnings

**Edit:**
- `internal/websterengine/summary.go`
- `internal/websterengine/summary_test.go`

**Intent:**
Add the summary-document half of D5's recorded warning, after the pattern of `AppendIntegrationTriage`.

`summary.go` gains `AppendAuditWarnings(websterDir string, warnings []string) error`.
It is a no-op for an empty list, and otherwise appends, through `appendToSummary`, a section headed `## Audit warnings`, a one-sentence explanation that these findings (fork-audit policy findings, and drift about a later card, D9) were recorded as warnings and did not stop the run, and one bullet per warning in the order given.
Update the file header's list of write-side helpers.

`summary_test.go` covers: empty list → file byte-identical; two warnings → the section appended after the existing content with both bullets in order; a missing summary file → an error naming the path, like the sibling appenders.

**ImpactSummary:** One new appender beside the existing ones; no current caller until card 15.
