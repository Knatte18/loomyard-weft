# Card 10 — cancel-marker-retire

**Delete:**
- `internal/battencli/cancelmarker.go`
- `internal/battencli/cancelmarker_test.go`
- `internal/shedrun#CancelMarker`
- `internal/shedrun#CancelMarkerLock`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun/doc.go`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#strandedEnded`
- `internal/dobbyengine#daemon.readWaits`
- `internal/dobbyengine#daemon.runNotices`
- `internal/battencli#DobbyRuns`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestStrandedEnded`
- `internal/dobbyengine#TestWaitReminders`
- `internal/dobbyengine#TestRunNotice_Conditions`
- `internal/dobbyengine/doc.go`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`

**Intent:** Retire the cancel marker, whose last callers card 9 removed.

Delete `cancelmarker.go` with its test file, and `shedrun.CancelMarker` and `shedrun.CancelMarkerLock`, whose path `shedrun.CancelLock` now names.
Drop their rows from `TestPathConstructors` and their sentences from `shedrun/doc.go`.
Rename `RunView.CancelMarker` to `RunView.Cancelling`, the run's batten record carrying a cancel intent without a finish,
in its declaration, in `DobbyRuns`, in every dobby reader listed here, in their tests' fixtures and in `dobbyengine/doc.go`'s mentions of the marker.
`TestCancelIntegration_BlockedRunWithNoPair` loses only its check that the marker file is gone after a finished cancel, since the file has no path left to check.
Grep for `CancelMarker`, `cancelMarker` and `cancel marker` afterwards and leave none in Go code, test names or package docs.

**ImpactSummary:** Deletes the marker file pair and two shedrun path functions with no callers left, and renames one dobby view field across its readers.

**Commit:** 10: cancel-marker-retire
