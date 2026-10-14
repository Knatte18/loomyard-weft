# Card 7 — startup-ready-from-events

**Create:**
- `plan:internal/shuttleengine#turnStartAfterSessionStart` -> `func turnStartAfterSessionStart(engine Engine, eventsPath string) (bool, error)`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted(started *bool)`

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, eventsProof bool) (Outcome, error)`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Intent:**
Decision D1: `start`'s startup probe reads readiness from the run's events ahead of the pane capture.

`checkLivenessTick` gains `eventsProof bool`.
`awaitStartup`, called only from `start`, passes true; `Run.Wait`, which runs the probe for Wait and attach, passes false, so a not-yet-started run seen by Wait or attach keeps today's capture rule.
With `eventsProof` true and `run.state.RecordsSessionStarts` set, after the liveness, session-gone and already-started checks and before `CapturePane`, the tick calls `turnStartAfterSessionStart`; a true answer calls `markStarted` and returns not-ready-yet with started set, as a ready capture does.
A false answer or a read error falls through to today's capture path unchanged, so a recognized gate is still captured and dismissed.
An engine whose launch does not record session starts never reaches the events read.

`turnStartAfterSessionStart` reads the events file from its start through the engine's `SessionSignalParser` and answers true when a `SessionSignalTurnStart` follows a `SessionSignalSessionStart`; an absent file reads false with no error, an unreadable one returns its error, and an engine without the parser reads false.
It reads no Claude hook name (`PATTERN-shuttle-provider-seam`).
`markStarted` is the ready-capture branch's body moved into one method: set `*started`, set and persist `run.state.Started` under `recordMu`, warning on a save failure; the ready-capture branch calls it too, so both answers set and persist `Started` the same way.
Bound: the proof is read only in `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it; a trust or bypass gate holds the first turn, so no turn start reaches the file while a gate is on screen; every negative answer still goes through `classifyStartupWindow` and the other exits that consult `allOutputFilesExist` first, since this change only adds a positive answer (`PATTERN-completion-signal`).
After card 6 every launch through `start` carries a first turn, so the proof is available on every `start`.

`awaitStartup`'s and `checkLivenessTick`'s doc comments and `internal/shuttleengine/doc.go`'s startup paragraph state the events-first answer and its bound.

Rows of `TestStartup_NeedsSessionStartLine`, which drives `StartGated` over `sessionFakeEngine` (`SESSIONSTART…` is a session start, `START` a turn start) and asserts `Started` persisted in `run.json` for every started row:
- new: a session start then a turn start, with the engine classifying every capture `StartupPending` (a blank pane), starts the run;
- changed: the existing row with a session start, no turn start and a pending pane now also asserts the start fails at the startup deadline with outcome `died`;
- kept: the existing row "a gate on screen is dismissed beside a session-start line, then the ready pane starts the run" already carries the criterion that a trust gate with no turn start recorded is dismissed and not reported ready, since a gate wrongly answered ready would start the run without the Enter the row asserts; no new gate row is added;
- new: a turn start before the session start, with a pending pane, is not started;
- new: a launch that does not record session starts, with a session start, a turn start and a pending pane, is not started.

`TestRun_Wait_StartupWindow` pins that Wait's and attach's probe keeps the capture rule: a new row "a session start then a turn start on a recording launch still needs a ready pane" seeds the events file with `SESSIONSTART:startup` then `START`, sets `recordsSessionStarts`, and scripts `StartupPending` on every capture, expecting `OutcomeDied` at the startup window.
The row runs over `sessionFakeEngine`, which reads those lines as signals, in place of the table's plain `fakeEngine`, so it fails if `Run.Wait` passed `eventsProof` true; the table gains an `events string` field the row writes to the events path before `Wait`, and a `sessionSignals bool` field selecting that engine.

**ImpactSummary:** `checkLivenessTick` has two callers, `awaitStartup` and `Run.Wait`, both in `wait.go`; only `start`'s probe gains the events-first positive answer, and Wait's and attach's probe are unchanged.
