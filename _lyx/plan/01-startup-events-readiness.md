# Card 1 — Start's startup probe reads readiness from the events file first

**Edit:**
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, eventsReady bool) (Outcome, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine#turnStartedAfterSessionStart` -> `func turnStartedAfterSessionStart(engine Engine, eventsPath string) bool`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted()`

**Intent:**
Decision D1: in `start`'s probe, a launch is ready once its events file holds a session-start signal and, after it, a turn-start signal, whatever the pane capture shows.

`turnStartedAfterSessionStart` (in `wait.go`) reads the whole events file through the engine's `SessionSignalParser` and reports true when a `SessionSignalTurnStart` follows a `SessionSignalSessionStart` of any source.
It never names a Claude hook, per `PATTERN-shuttle-provider-seam`.
An engine without the parser, an absent file and an unreadable file all answer false, so the capture path decides as today.

`checkLivenessTick` gains `eventsReady`.
Placement: after the `sessionGone` check and the `*started` short-circuit, before `CapturePane`.
When `eventsReady` and `run.state.RecordsSessionStarts` both hold and `turnStartedAfterSessionStart` answers true, the tick sets `*started`, clears `run.startupHookReason`, calls `markStarted` and returns `("", nil)` without capturing.
Otherwise the tick runs the capture path unchanged: a recognized gate is still dismissed, and a ready pane still needs the session-start line.
`markStarted` is the existing persist block moved out of the `StartupReady` arm: set `run.state.Started` under `recordMu`, save, and log a failed save at Warn, non-fatal as today.
Both ready arms call it, so an events-ready start persists `Started` exactly as a ready capture does.
`awaitStartup` passes `true`.
`Wait`'s own startup probe passes `false`: Wait and Attach keep the capture rule, because a re-attached run's events file may hold an earlier incarnation's lines.
An engine whose launch does not record session starts keeps the capture-only probe.
The change adds only a positive answer and no return statement yielding a negative verdict.
So `TestCompletionSignal_NegativeVerdictReturnSites` and `TestCompletionSignal_FileContractCallSites` stay as they are (`PATTERN-completion-signal`).
Bound: the events proof is read only by `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it.
A trust or bypass gate holds the first turn, so no turn start reaches the file while a gate is on screen.

Doc comments: `awaitStartup`'s, `checkLivenessTick`'s ("For a record with RecordsSessionStarts …" paragraph) and `Wait`'s call site say which probe reads the events proof.
`doc.go`'s startup paragraph (the `Launch.RecordsSessionStarts` sentences under "Start/StartGated/Run/RunGated run the startup probe") gains the start-only events proof and says Wait's probe keeps the capture rule.

Tests, all rows of existing tables:
- `TestStartup_NeedsSessionStartLine` (its engine is `sessionFakeEngine`, where `SESSIONSTART:<source>` is a session start and `START` a turn start).
  Add these rows:
  - "a session start then a turn start starts the run while the pane stays pending": `records: true`, script `[StartupPending]`, events `"SESSIONSTART:startup\nSTART\n"`, `wantStarted`.
    It fails if the probe still needs a ready capture.
  That row also gets a field `wantNoCapture`, asserting `reed.CallLog` holds no `"CapturePane"`, so the events check must run before the capture.
  The test's doc comment gains the events-only start.
  The table's `wantStarted` branch also loads `run.json` through `loadRunState(run.RunDir())` and asserts `Started`, as `TestStartup_TrustPromptThenReady` does, so an events-ready start that skipped `markStarted` fails.
  - "a turn start before the session start is not started": `records: true`, script `[StartupPending]`, events `"START\nSESSIONSTART:startup\n"`, not started.
  - "a trust gate with a session start and no turn start is dismissed and not started": `records: true`, script `[StartupTrustPrompt]`, events `"SESSIONSTART:startup\n"`, `wantEnter`, not started.
  - "a launch that does not record session starts is not started on events": `records: false`, script `[StartupPending]`, events `"SESSIONSTART:startup\nSTART\n"`, not started.
  The existing row "a session-start line of an earlier launch with a pane not yet ready is not started" already pins a session start with no turn start and a pending pane dying at the startup deadline.
- `TestRun_Wait_StartupWindow`: add a field `events string`.
  When it is set, the row seeds that content into the run's events file and runs over a `sessionFakeEngine`, whose zero `liveness` is `LivenessUnproven` so `sessionGone` stays false, instead of the bare `fakeEngine`.
  Add the row "a session start then a turn start on a recording launch does not start Wait's probe": script `[StartupPending]`, `recordsSessionStarts: true`, events `"SESSIONSTART:startup\nSTART\n"`, `wantOutcome: OutcomeDied`.
  The trust-dismiss assertions keep reading the `fakeEngine` that the `sessionFakeEngine` embeds.

**ImpactSummary:** `checkLivenessTick` gains one parameter, with both callers in `wait.go`; the events-ready answer is reachable only from `start`'s probe, and Wait's and Attach's probes are unchanged.

**Commit:** 1: shuttle: start's startup probe reads readiness from the events file first
