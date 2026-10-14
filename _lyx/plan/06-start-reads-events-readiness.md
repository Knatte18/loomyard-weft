# Card 6 — start-reads-events-readiness

**Create:**
- `plan:internal/shuttleengine#startupTurnStarted` -> `func startupTurnStarted(engine Engine, eventsPath string) bool`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted()`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#sessionStartRecorded`

**Intent:** D1: `start`'s startup probe reads readiness from the run's events ahead of the pane capture, so a collapsed or blank pane no longer holds back a provider that is already past every startup gate and working.

A new `Run` field, `eventsReadiness bool` (a struct field, so no arrow), marks a probe that may answer ready from events.
`awaitStartup`, which only `start` calls, sets it to `run.state.RecordsSessionStarts` before its loop; `Wait`'s own startup probe, which an attach of a not-yet-started run runs, never sets it, so Wait and attach keep today's capture rule over an events file that may hold an earlier incarnation's lines.
Only `start`'s probe reads the proof, and its events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it.

`startupTurnStarted` reads the events file through the engine's `SessionSignalParser` and reports whether it holds a session-start signal and, after it in file order, a turn-start signal; an engine without the parser, an absent file and an unreadable file read false.
It reads signal kinds only, never a Claude hook name, per `PATTERN-shuttle-provider-seam`.

In `checkLivenessTick`, after the liveness and `sessionGone` checks and the `*started` short-circuit, and before the pane capture: when `run.eventsReadiness` is set and `startupTurnStarted` answers true, the tick sets `*started`, calls `markStarted` and returns `("", nil)` without capturing the pane.
`markStarted` is today's ready-capture persistence block moved into a method (lock `recordMu`, set `run.state.Started`, `saveState`, Warn on a failed save), and the ready-capture branch calls it too, so an events answer sets and persists `Started` exactly as a ready capture does.
A launch the events do not yet prove ready falls through to today's capture path unchanged, so a recognized gate is still dismissed and the session-start-line rule for a ready pane still holds.
Bound, and what the change lets through: the events proof adds only a positive answer; every negative startup answer is still reached through the capture path, `classifyStartupWindow` and `classifyDeadlineExpiry`, which consult `allOutputFilesExist` first (`PATTERN-completion-signal`).
A trust or bypass gate holds the first turn, so no turn start reaches the events file while a gate is on screen, and an engine whose launch does not record session starts keeps the capture-only probe.
Update the doc comments of `awaitStartup` and `checkLivenessTick` to state the events answer and its bound.

Rows added to `TestStartup_NeedsSessionStartLine`, which runs through `StartGated` over `sessionFakeEngine` (`SESSIONSTART:<source>` a session start, `START` a turn start); each pane-pending row sets `CaptureQueue` to `[]string{""}` so every capture is blank:
- "a session start then a turn start start the run while every capture is blank": `records`, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, `wantStarted`.
- "a turn start read from events starts the run without dismissing a gate the capture would show": `records`, script `StartupTrustPrompt`, events `SESSIONSTART:startup\nSTART\n`, `wantStarted`, no Enter; it fails if the probe captured before reading events.
- "a trust gate with no turn start recorded is dismissed and not reported ready": `records`, script `StartupTrustPrompt`, events `SESSIONSTART:startup\n`, `wantEnter`, not started.
- "a turn start before the session start does not start the run": `records`, script `StartupPending`, events `START\nSESSIONSTART:startup\n`, not started.
- "a launch that does not record session starts ignores the events proof": `records` false, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, not started.
The existing row "a session-start line of an earlier launch with a pane not yet ready is not started" gains `wantErr` set to `outcome "died"`, pinning that a session start with no turn start and blank captures stays pending until the startup deadline ends it died.
For every `wantStarted` row the test also asserts that the persisted `run.json` reads `Started: true`, which fails if the events answer skipped `markStarted`.

`TestRun_Wait_StartupWindow` gains a row field `events string`; a row with events runs over a `sessionFakeEngine` seeded with the row's script in place of the plain `fakeEngine`, and writes `events` to the run's events path before `Wait`.
New row "a session start and a turn start on a recording launch do not start Wait's probe": `startupScript` `StartupPending`, `recordsSessionStarts` true, events `SESSIONSTART:startup\nSTART\n`, `wantOutcome: OutcomeDied`; it fails if Wait's probe took the events answer.

Docs: `internal/shuttleengine/doc.go`'s startup paragraph (the `Launch.RecordsSessionStarts` sentence) adds that `start`'s probe answers ready, ahead of the capture, once the fresh events file holds a session start and then a turn start, while Wait's and attach's probe keep the capture rule.

**ImpactSummary:** shuttle's startup probe in wait.go gains an events-first ready answer for start only, through a new Run field; two startup tests gain rows; shuttleengine doc.go gains a sentence.

**Commit:** 6: Read start's readiness from the events file before the pane capture
