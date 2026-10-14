# Card 6 — `start`'s startup probe reads readiness from the events first

**Create:**
- `plan:internal/shuttleengine#turnStartedAfterSessionStart` -> `func turnStartedAfterSessionStart(engine Engine, eventsPath string) (bool, error)`

**Edit:**
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, eventsProof bool) (Outcome, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Intent:** D1, under `PATTERN-shuttle-provider-seam` and `PATTERN-completion-signal`.

`turnStartedAfterSessionStart` (in `internal/shuttleengine/wait.go`, beside `sessionStartRecorded`) reads the events file from its start through the engine's `SessionSignalParser` and answers true when a `SessionSignalSessionStart` is followed, later in file order, by a `SessionSignalTurnStart`.
An absent file answers false with no error, an unreadable one returns the read error, and an engine without `SessionSignalParser` answers false.
It names no provider hook; the signals come only from the engine's parser.

`checkLivenessTick` gains `eventsProof`.
When it is true and `run.state.RecordsSessionStarts` holds, the tick checks `turnStartedAfterSessionStart` after the liveness and `sessionGone` checks and before it captures the pane; a true answer sets `*started`, clears `run.startupHookReason` and persists `Started` exactly as the `StartupReady` branch does (move that persist into one place both branches use), and returns `("", nil)`.
A false answer or a read error falls through to today's capture path unchanged, so a recognized trust or bypass gate is still dismissed and a ready pane still needs its session-start line.
`awaitStartup` passes true; `Wait`'s startup probe (the attach of a not-yet-started run, whose events file may hold an earlier incarnation's lines) passes false and keeps the capture rule.
Bound: the proof is read only in `start`'s probe, whose events file was created fresh in this launch's new run directory, so only this launch's hooks can satisfy it; a gate holds the first turn, so no turn start reaches the file while one is on screen; an engine that does not record session starts keeps the capture-only probe.
This adds only a positive answer: every negative answer still goes through `classifyStartupWindow`, `classifyDeadlineExpiry` or `allOutputFilesExist` as `PATTERN-completion-signal` requires.

Tests, rows of `TestStartup_NeedsSessionStartLine` (`internal/shuttleengine/startup_test.go`), whose `sessionFakeEngine` reads `SESSIONSTART:<source>` as a session start and `START` as a turn start, and whose `CaptureQueue` capture `"gate"` is classified by the row's `StartupScript`:
- `a session start then a turn start start the run while every capture reads pending`: records, script `[StartupPending]`, events `SESSIONSTART:startup\nSTART\n`, want started and no Enter.
- The existing row `a session-start line of an earlier launch with a pane not yet ready is not started` already holds the inputs of "a session start, no turn start, blank captures": it is renamed `a session start with no turn start and pending captures dies at the startup deadline`, since `start`'s events file is fresh, and gains `wantErr: outcome "died"` beside its `wantNoErr`.
- `a trust gate with a session start and no turn start is dismissed and not ready`: records, script `[StartupTrustPrompt]`, events `SESSIONSTART:startup\n`, want Enter sent and `ErrNotStarted`.
- `a turn start before the session start does not start the run`: records, script `[StartupPending]`, events `START\nSESSIONSTART:startup\n`, want `ErrNotStarted`.
- `a launch that does not record session starts ignores a turn start`: no records, script `[StartupPending]`, events `SESSIONSTART:startup\nSTART\n`, want `ErrNotStarted`.
The first row fails while the probe reads only the capture; the other rows fail if the proof is read loosely.
For every `wantStarted` row the table's loop also reads `loadRunState(run.RunDir())` and requires `Started` true, so an events-ready answer that skips the persist fails.

`TestRun_Wait_StartupWindow` (`internal/shuttleengine/wait_test.go`) gains an `events string` field: when set, the row writes it to the events file and runs over a `sessionFakeEngine` with the row's `StartupScript` instead of the plain `fakeEngine`.
New row `a session start then a turn start on an attached not-started run keeps the capture rule`: `recordsSessionStarts`, script `[StartupPending]`, events `SESSIONSTART:startup\nSTART\n`, want `OutcomeDied`; it fails if `Wait` passes `eventsProof` true.
No new top-level test.

`internal/shuttleengine/doc.go`: the startup paragraph says `start`'s probe answers ready once its fresh events file holds a session start and then a turn start, before it captures the pane, persisting `Started` as a ready capture does, while the attach probe keeps the capture rule.

**ImpactSummary:** One unexported method re-signed with two callers in wait.go (awaitStartup, Wait), one new reader beside sessionStartRecorded; only shuttle's start path gains a ready answer, and attach's probe is unchanged.

**Commit:** 6: shuttle: start's startup probe reads a turn start from the events
