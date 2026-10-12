# Plan round 1 — fixer report

Every finding's premise held against the code and the cards; all eight are fixed as proposed.
`lyx loom validate-plan` passes after the changes, with informational findings only.

## Changes

- **F1** — `lyx loom insert-card 10 gateslot-watchdog-line` added card 10 and moved cards 10 to 13 to 11 to 14.
  The new card creates `internal/gateslot/watchdogline.go`, its test file, `WatchdogLine`, `FindWatchdogQuit`, `BusyQuitWayForward` and `TestWatchdogLine` (a table: the exact line, the line amid output before a `SIGQUIT: quit` header, and a log with no line), carries the moved Intent paragraph, edits `internal/gateslot/doc.go` with a "Watchdog line" section, and lists `cmd/lyx/testdata/test-budget.yaml`, raising gateslot's row from 11 to 12.
  Card 11 (the old card 10) drops the three gateslot Create targets and the `watchdogline.go` paragraph, lists the three symbols under Uses, and keeps TestWatchdog's assertion against `gateslot.WatchdogLine`.
  The prose mentions insert-card reported are retargeted: cards 8 and 9 now name card 11 as the watchdog card.
- **F2** — card 12 (the old card 11) gains a Bound paragraph after the no-hang sentence: markers match as text anywhere in the log, any exit code of 128 or above reads as a hang, and a match skips only the one rerun.
  Its `websterengine` package-doc sentence states that bound beside D5's hang set.
- **F3** — card 2 adds Edit targets `internal/loomengine#CheckSeed`, `internal/loomcli#poisonStatusFileMalformed` and `internal/loomshed#TestSeed_RefusesUndecodableFileAsExists`, and its list of comments the change makes wrong covers CheckSeed's inline comment and doc, poisonStatusFileMalformed's contrast, and the test's doc and its "unknown top-level field" row name.
- **F4** — card 4 states that `TestReadJSONStrict_MissingFile_NoMkdirAll` is renamed by hand to `TestReadJSONNoMkdir_MissingFileCreatesNoDirectory`, its declaration and every mention, before `replace_all` runs on `internal/state/strict_test.go`; the content-field sentence stays.
- **F5** — card 12 keeps `failTimedOut` as the path; `renderVerifyGateFindings`'s timed-out sentence and `verifyGateStuckReason`'s timed-out case now say the step was not re-run.
  The stuck-reason test's timeout row also wants "not re-run", and `TestRenderVerifyGateFindings` adds a timed-out report row asserting it.
- **F6** — card 11's watchdogApplies bullet no longer lists a hand-set value as ignored, says any value naming a live ancestor skips the watchdog, a hand-set one naming the invoking shell or pid 1 included, and the Bound paragraph lists that run among those bounded only by go test's own `-timeout`.
- **F7** — card 11's TestWatchdog gains a row whose sampler fails once with a plain error and then reports a busy `.test` process for the window, asserting sampling continues, one signal, and the failure logged in the `logcapture.CaptureVerbose` buffer.
- **F8** — card 8's refusal text, in the decodeConfig bullet and the TestLoadConfig row, reads `want at least 0`.

## Disputed

None.

## Deferred

None.
