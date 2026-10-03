# Card 10 — anomaly-pending-retry

**Edit:**
- `internal/loomcli/selfreport.go`
- `internal/loomcli/selfreport_test.go`
- `internal/loomcli/selfreport_github_test.go`
- `internal/loomengine/config.go`

**Uses:**
- `internal/loomengine#RenderAnomalyBody`
- `internal/selfreportengine#DefaultLabels`

**Intent:**
A Tier 1 anomaly whose filing failed is kept as a pending entry in the filed-title marker and retried on every later detection pass, so a halt that is resumed before its filing succeeds is still filed (PR-Gate F2, operator addition (b)).
Today a failed `FileIssue` leaves the title unrecorded "to retry next run", but detection reads only the final status, so once the run is resumed past the halt the anomaly is never detected again and never filed.

`selfreportFiledMarker` gains `Pending []pendingAnomaly` (`json:"pending,omitempty"`), where `pendingAnomaly` holds the anomaly's `Title` and its rendered `Body`.
A marker written before this change reads as one with no pending entries.
A title is in at most one of `Titles` and `Pending`.

`runFilingPass` in order:

1. Read the marker; return without a write when it holds no pending entry and there is no new anomaly.
2. Retry every pending entry in recorded order through `deps.FileIssue` with its stored body and `selfreportengine.DefaultLabels()`.
   A success moves its title from `Pending` to `Titles` and writes the marker at once; a failure warns and leaves the entry pending.
3. Collapse the new anomalies by title as today and skip a title already recorded or pending.
   A success records the title and writes the marker; a failure appends the title and `loomengine.RenderAnomalyBody`'s body to `Pending` and writes the marker at once.

`detectAndFileAnomalies` keeps its three skips (knob off, `ErrShedBusy`, cancelled context), so a pass any of them skips retries nothing.
Past them, every path reaches the filing pass: a status file that is missing, unreadable or carries an undecodable product now calls `runFilingPass` with no new anomalies instead of returning, so its pending entries are still retried.
The cancelled-context arm is unchanged: it runs the filing pass only for a lone crash-resume, which then also retries pending entries.
`runFilingPass`'s, `selfreportFiledMarker`'s and `detectAndFileAnomalies`' doc comments state the pending list and the retry, and drop "retried next run".
`loomengine.LoomSelfreportFiled`'s doc states that the marker also holds the anomalies whose filing failed and awaits a retry.

What this lets through, and the bound: a pass now files an anomaly it did not detect in this pass, but only one an earlier pass detected and failed to file, under that pass's title and body.
Each pending title is filed at most once per pass and leaves the list on its first success; the knob, busy and cancelled-context skips bound retries as they bound detection.
A `FileIssue` that created the issue but returned an error files it again on the next pass, the same duplicate today's retry already accepts.

New `selfreport_test.go` cases, over the existing `selfreportTestFixture`:

- the done-when case: a `failed` final status and a failing filer leave the producer-hard-failure title pending with its rendered body and unrecorded; the status is then rewritten to `running` with the halt resumed past, and a later pass with a working filer files that title once with the stored body and `DefaultLabels()`, after which the marker records it and holds no pending entry, and a third pass files nothing;
- a pending entry whose retry fails again stays pending, and the pass still files a newly detected anomaly;
- a pending title that the same pass detects again is filed once;
- a missing status file, an unreadable status file and an undecodable product each still retry the pending entries;
- the knob off, a busy run error, and a cancelled context with no crash-resume each leave the pending entries in place and call no filer;
- a cancelled context with a crash-resume entry observation files the crash-resume and retries the pending entries;
- the marker round-trip test also round-trips a pending entry, and a marker file holding only `titles` reads with no pending entries.

`selfreport_github_test.go`'s `TestDetectAndFileAnomalies_FilingFailure_LeavesMarkerUnfiledAndDoesNotPropagate` keeps its name; its doc says a failed title stays out of `Titles` and is kept pending, and it also asserts the pending entry holds that title and the body the engine boundary was sent.

**ImpactSummary:** confined to `internal/loomcli`'s Tier 1 filing pass and its marker, which only `detectAndFileAnomalies` reads and writes, plus one doc comment in `loomengine`; the marker file stays readable across the change.
