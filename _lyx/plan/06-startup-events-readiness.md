# Card 6 — startup-events-readiness

**Edit:**
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, launchProbe bool) (Outcome, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine#launchTurnStarted` -> `func launchTurnStarted(engine Engine, eventsPath string) bool`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted()`

**Intent:**
D1: `start`'s startup probe reads readiness from the run's events ahead of the pane capture.

`launchTurnStarted` reads the events file whole through the engine's `SessionSignalParser` and reports true when a `SessionSignalSessionStart` signal is followed, later in the file, by a `SessionSignalTurnStart` signal.
An engine without the parser, an absent file and an unreadable file read false, so the probe falls back to the capture.
It reads signals by kind only, never by a Claude hook name, per `PATTERN-shuttle-provider-seam`.

`checkLivenessTick` takes a new `launchProbe` parameter.
After the liveness checks, `sessionGone` and the `*started` short-circuit, and before `CapturePane`, a call with `launchProbe` set on a record with `RecordsSessionStarts` asks `launchTurnStarted`; a true answer clears `startupHookReason`, sets `*started` and calls `markStarted`, and returns without capturing the pane.
Otherwise the tick goes on to the capture exactly as today, so a recognized gate is still dismissed and a ready pane still needs its session-start line.
`markStarted` is today's persistence of `Started`, under `recordMu` with its non-fatal Warn, moved out of the `StartupReady` branch, which now calls it too; the events answer therefore persists `Started` exactly as a ready capture does.
This adds only a positive answer: every negative answer still goes through `classifyStartupWindow` and the output-file checks, per `PATTERN-completion-signal`.

`awaitStartup`, which only `Runner.start` calls, passes `launchProbe` true; its doc comment states the events answer and its bound.
`Wait` passes false, so Wait's and attach's probe of a run not yet started keeps today's capture rule.
Bound: the events answer is read only by `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it; a trust or bypass gate holds the first turn, so no turn start reaches the file while a gate is on screen; an engine whose launch does not record session starts keeps the capture-only probe.

`TestStartup_NeedsSessionStartLine` gains a per-row `capture` field (default `"gate"`, the value its queue holds today) and these rows, each over `sessionFakeEngine`, whose parser reads `SESSIONSTART:<source>` as a session start and `START` as a turn start:
- `a session start then a turn start start the run while every capture is blank`: records, script `StartupPending`, capture `""`, events `SESSIONSTART:startup\nSTART\n`, wants started;
- `a turn start before the session start is not started`: records, script `StartupPending`, capture `""`, events `START\nSESSIONSTART:startup\n`, wants not started;
- `a gate on screen with no turn start is dismissed and not started`: records, script `StartupTrustPrompt`, events `SESSIONSTART:startup\n`, wants Enter sent and not started;
- `a launch that does not record session starts ignores a turn start`: not recording, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, wants not started.
The existing row `a session-start line of an earlier launch with a pane not yet ready is not started` becomes `a session start with no turn start and blank captures dies at the startup deadline`: it sets `capture` to `""` and adds `wantErr: outcome "died"`, as the process-death row does, so a timeout outcome or a start fails it.
The started rows also assert that run.json reads `Started` true, as `TestStartup_TrustPromptThenReady` does.

`TestRun_Wait_StartupWindow` gains an `events` field written to its events path and a row `a Wait probe ignores an earlier launch's session and turn start`: records, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, wanting `OutcomeDied`; the row runs over `sessionFakeEngine` so the parser is present and only `launchProbe` false keeps it from starting.

`internal/shuttleengine/doc.go`'s startup paragraph states that `start`'s probe treats a session start followed by a turn start in the events file as ready ahead of the capture, and that Wait's probe does not.

**ImpactSummary:** `checkLivenessTick` gains a parameter set only by `awaitStartup`; its two callers are both in `wait.go`, and Wait's behavior is unchanged.
