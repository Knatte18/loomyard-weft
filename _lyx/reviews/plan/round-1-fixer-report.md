# Plan round 1 — fixer report

Every finding's premise was checked against the code and holds.
`lyx loom validate-plan` answers ok after the fixes.

## Changes

- F1, fixed as proposed: `00-overview.md`'s untagged and `--tags integration` verify lines add `./tools/sandbox` before `./cmd/lyx`; `tools/sandbox/suite.go` embeds the three suites card 6 edits, and `TestRunSuite_PerSuiteSpec` reads them.
- F2, fixed as proposed: card 4's exit-path table adds the stop-with-an-idle-listing-and-a-cancelled-context row, and the Intent names it as the test of the nil branch's `cmd.Context()` guard.
- F3, fixed as proposed: card 5's Intent drops the `c.hubPath` non-empty condition, says every reed CLI engine is built from `hubgeom.ReedGeometry` with `c.hubPath` set (`internal/reedcli/cli.go` sets it beside the geometry in `PersistentPreRunE`), and points the standalone criterion at card 3's no-path-told row; the empty-`hubPath` row is deleted.
- F4, fixed as proposed: card 3 drops the `TestTouchDiscoverSignal_TruncatesAPendingStop` Create bullet and paragraph, and its last-session row now calls `touchDiscoverSignal` and asserts the file exists and is empty.
- F5, fixed as proposed: card 4 limits the empty-logs-directory assertion to the lock-already-held row and has every other row assert its own trace lines, with the stop-with-an-idle-listing row's exit-on-stop line and the no-successor rows' absent successor-spawned line as examples.
- F6, fixed as proposed: card 4's stop-with-an-affirmative-listing row waits for two lister calls after the marker is consumed, the second proving the stop-woken cycle finished.

## Disputed

None.

## Deferred

None.
