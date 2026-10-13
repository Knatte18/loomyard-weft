# Card 3 — plain-overview-panes

**Edit:**
- `internal/statuspane#Frame`
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane/doc.go`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#battenCLI.overviewCmd`
- `internal/battencli#TestRenderOverview`

**Delete:**
- `internal/statuspane#ReportFromEnv`
- `internal/statuspane#fitLines`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/battencli#fitOverview`
- `internal/battencli#overviewMoreRunsFormat`

**Create:**
- `internal/statuspane#TestPane_Run`

**Uses:**
- `internal/selfexec#New`
- `internal/selfexec#Watcher.Check`

**Intent:** Both overview panes stop reporting a content size and stop cutting their frame (D7's first half), and the shared pane loop re-execs on a changed binary (D8's wiring).

`statuspane.Frame` loses its `Fit` field and keeps `Lines`.
`statuspane.Pane` loses its `Report` field and gains `Check func()`, the binary check run once per poll.
`Pane.Run` drops the report step and `fitLines`: it redraws every line of the frame, uncut, so a frame taller than the terminal scrolls its first lines into the history and a line wider than the terminal wraps.
It still reads `Size`, and a size change alone still redraws, so a wrapped frame is redrawn for the new width.
Each poll, before reading the frame, it calls `Check`; a nil `Check` means `selfexec.New().Check`, built once when `Run` starts, the same nil-means-production idiom `After` uses.
The check adds no wake: it runs on the poll's own wait, which keeps its one-second floor and eight-second ceiling (`PATTERN-daemon-wakeups`).
`ReportFromEnv` and `fitLines` are deleted with their last callers.
`internal/statuspane/doc.go` drops the report, fit and cut paragraphs, states that every line is drawn uncut and that a re-exec keeps the pane, its strand binding and `TMUX_PANE`, and its import list gains `internal/selfexec` (the Constraints' doc line) and drops `internal/agentname`, whose one production use was `ReportFromEnv`.

`checklistCmd` drops `Report` from its `Pane` and with it the `reedengine.LoadConfig`, `hubgeom.ReedGeometry` and `reedengine.New` lines that only fed `SetContentRows`; its `Long` text and doc comment stop saying it reports a line count and describe card 2's frame: detail blocks, the row list, the state line and the wait line.
`overviewCmd` drops `Report` and the reed config, geometry and engine lines the same way, and its `Long` text and doc comment stop saying it reports a line count.
`overviewSource.frame` returns `Lines` only; `fitOverview` and `overviewMoreRunsFormat` are deleted, and `TestRenderOverview` loses its `fitRows`/`wantFit` fields and the two rows that used them, plus the file comment's mention of `fitOverview`.
After this card no overview pane calls `SetContentRows` or draws a "not shown" line: removing `Pane.Report` closes the report seam, and the card's `Verify:` fails when either verb's file still names `SetContentRows` or a "not shown" line.

`TestPane_RunAndReportFromEnv` is deleted and `TestPane_Run` takes its table, keeping the redraw, back-off, size-change and resize rows and changing the rest:
"FrameTallerThanTheTermRowsPrintsFitRowsAndNeverMore" becomes "FrameTallerThanTheTermRowsIsDrawnWhole", every line redrawn;
"LineWiderThanTheColumnsIsCut" becomes "LineWiderThanTheColumnsIsDrawnUncut";
"FailedSizeKeepsEveryLineUncut" stays;
the two report rows, the `wantReports` and `failReports` fields and the `ReportFromEnv` subtest go;
a new field `wantChecks` with every row asserting `Check` ran once per poll, so a loop that skips the check fails.
`internal/statuspane`'s budget row stays at 6: one top-level test leaves and one arrives.

**ImpactSummary:** Two pane verbs lose their `SetContentRows` wiring and fit helpers, and `statuspane`'s exported surface loses `ReportFromEnv` and `Frame.Fit`; those two verbs are the only importers of either, and no reed code changes.

**Verify:** ! grep -n "SetContentRows\|not shown" internal/loomcli/checklist.go internal/battencli/overview.go
