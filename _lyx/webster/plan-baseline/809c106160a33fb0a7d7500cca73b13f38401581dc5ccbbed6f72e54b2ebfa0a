# Card 11 — drive-handback

**Create:**
- `internal/shedrun#BattenWatchedContent`
- `internal/shedrun#ParseBattenWatched`
- `internal/shedrun#TestBattenWatchedContent`
- `internal/loomcli#battenWatch`
- `internal/loomcli#readBattenWatch`
- `internal/loomcli#driveNoticeTail`
- `internal/loomcli#driveHandBack`
- `internal/loomcli#driveNotices`
- `internal/loomcli#driveReport`
- `internal/loomcli#driveReport.render`
- `internal/loomcli#driveParkDeps`
- `internal/loomcli#handBackRun`
- `internal/loomcli#commitAndPark`
- `internal/loomcli/drivehandback_test.go`

**Edit:**
- `internal/shedrun#BattenWatchedFileName`
- `internal/shedrun/doc.go`
- `internal/loomcli#driverWatched`
- `internal/loomcli#liveBattenPID`
- `internal/loomcli#loomCLI.commitRecordsCmd`
- `internal/loomcli#TestDriverWatched`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#writeParkMarker`
- `internal/loomcli#driverReportPath`
- `internal/loomcli#driveStop`

**Intent:**
D8 and D9: what every Go-driver hand-back leaves, which is a Go-authored stop report, the run records committed and the park marker written by the same code `lyx loom commit-records --park` runs, plus the parent messages the LLM driver's two notify rules send.
A new file `internal/loomcli/drivehandback.go` holds it.

`commitAndPark` is `commitRecordsCmd`'s `--park` body moved out unchanged: `commitRecordsVerb`, then `writeParkMarker` whatever the commit's outcome, a marker write failure failing the call.
`commitRecordsCmd` calls it, and the Go driver calls it with a discarding writer.

The batten-watched marker's format is declared once in `shedrun`, beside `BattenWatchedMarker`:
- `BattenWatchedContent` renders the pid in decimal on the first line, then, when non-empty, the prime's batten scratch directory and the Run-Shed row's name on the next two lines.
- `ParseBattenWatched` reads the first line as the pid and the next two lines when present, so today's one-line marker parses with empty scratch directory and producer.
  A marker whose first line is not a positive pid is an error.
- `BattenWatchedFileName`'s comment says the file holds those lines.
Card 21 switches batten's writer to `BattenWatchedContent`.

`battenWatch` is the marker's reading: `PID`, `ScratchDir` and `Producer`.
`readBattenWatch` parses the marker through `shedrun.ParseBattenWatched` and returns the parsed `battenWatch` whenever the marker parses, with the bool true only when `isAlive` confirms the pid.
So an unwatched driver still learns the prime a dead batten named (D4).
`driverWatched`'s doc comment says the marker's first line holds the pid, and `internal/shedrun/doc.go` names the two new marker functions.
`liveBattenPID` delegates to it, so `driverWatched` and `battenWatcher` read a three-line marker the same as today's one-line one.
This lands before batten writes the longer marker, so no driver ever misreads it.

`driveHandBack` describes one hand-back: the stop, the rules applied, the attempts with their verdicts, the parent question, the parent notice to relay, whether a `done` reported `friction: failed`, and `WatchedVerifyLine`.
`driveNotices` applies D8 and returns each message ending in `driveNoticeTail`:
- watched: the `parent_notice` verbatim, sent after any arbiter outcome at the stop; a line naming the run-id, the stop report and the question when the hand-back carries a question; a line naming the run-id and the stop report at a `friction: failed` done; and, with `WatchedVerifyLine`, the unwatched line below (D4);
- unwatched: the same, plus one line naming the run-id and the stop report at every other hand-back.
`driveReport.render` writes the run-id, its position (history length and the step's progress), the envelope's `reason`, every rule applied, every tackler and arbiter attempt with its verdict and verdict path, the parent question when present, the friction and repair directories, and one line per send, failed send or unqueued send.

`driveParkDeps` carries `ReportPath`, `Commit commitStatusDeps`, `MarkerPath`, `Notify func(line string) error`, `OrchRecorded func() (bool, error)` and `Now`.
`handBackRun` sends each notice through `Notify` and records the outcome in the report.
With no orch strand recorded, it logs the line and queues nothing, and the report alone carries it.
It then writes the report at `ReportPath`, a fresh `driverReportPath`, creating its directory, and calls `commitAndPark`.
It returns a report-write failure; a commit failure is logged, and the park marker is still written, as `--park` does.
For a hand-back describing a `friction: failed` done, `handBackRun` writes the report at `ReportPath`, sends the notice through `Notify` under the `OrchRecorded` rule and records the outcome line, but calls no `commitAndPark`, so a finished run is never parked.
Releasing the driver lock and exiting are the caller's (card 12).

Tests:
- `TestBattenWatchedContent` in `internal/shedrun/paths_test.go`, a table: a pid alone renders one line and parses back with empty fields; a pid with a scratch directory and a producer round-trips; an empty file, a non-numeric and a zero pid each fail to parse.
- `TestDriverWatched` gains a row whose marker holds a live pid, a scratch directory and a row name on three lines, reading watched.
- `TestDriveNotices` in `drivehandback_test.go`, a table over watched and unwatched:
  - a `parent_notice` relay is the notice verbatim plus the tail;
  - a question hand-back names the run-id, report and question;
  - a `friction: failed` done notifies under both rules;
  - an awaiting hand-back without a notice sends nothing when watched and one run-id line when unwatched;
  - a watched verify halt sends the unwatched line;
  - every message ends with the tail.
- `TestDriveHandBack` in `drivehandback_test.go`, over fake `Notify`, `OrchRecorded` and commit seams in a temp worktree:
  - the report holds the run-id, reason, rules, attempts, question and directories;
  - a failed send and a no-orch-strand line each appear in the report, and no orch strand queues nothing;
  - the commit runs and the park marker holds the report path;
  - a failing commit still writes the marker;
  - a `friction: failed` done writes the report with its send line and writes no park marker.
`TestCommitRecordsVerb_*` keep pinning the commit envelope that `commitAndPark` wraps.
`test-budget.yaml`'s `internal/loomcli` row rises by two and its `internal/shedrun` row by one.

**ImpactSummary:** the `--park` body moves into a shared function with unchanged behavior, and the watched-marker read tolerates two more lines; every existing caller reads a one-line marker as before.
