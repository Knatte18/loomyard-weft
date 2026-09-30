# Card 6 — loom-driver-report-path

**Edit:**
- `internal/loomcli#driverReportPath`
- `internal/loomcli/driverreport_test.go`

**Uses:**
- `plan:internal/shedrun#DriveReportsDir`

**Intent:**
Write each ly-drive attempt's report into the durable run directory so it can be committed and survive teardown.
`driverReportPath` composes onto `shedrun.DriveReportsDir(l, runID)` instead of `shedrun.ScratchDir(l, runID)`, keeping the `drive-report-<timestamp>-<4-hex>.md` filename and both injected seams.
Rewrite its doc comment: the directory is the durable drive-reports directory, still composed through a shedrun accessor rather than by spelling a segment here.
`startDriver` already runs `os.MkdirAll(filepath.Dir(reportPath))` before launch, so the new directory is created there with no further change.

Tests: rename `TestDriverReportPath_LandsUnderRunScratchDir` to `TestDriverReportPath_LandsUnderDriveReportsDir` and assert the parent equals `shedrun.DriveReportsDir(l, runID)`; the other cases stay.

**ImpactSummary:** One path composer changes directory; its sole caller, the driver bootstrap, already creates the parent.
