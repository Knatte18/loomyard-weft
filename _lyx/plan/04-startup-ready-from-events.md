# Card 4 — startup-ready-from-events

**Create:**
- `plan:internal/shuttleengine#turnStartAfterSessionStart` -> `func turnStartAfterSessionStart(engine Engine, eventsPath string) bool`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted()`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`

**Intent:** Let `start`'s startup probe read readiness from the run's events ahead of the pane capture (D1).

`Run` gains a field `startupFromEvents bool`, set only by `awaitStartup` before its loop, so only `start`'s probe reads the events proof; `Wait`'s and `Attach`'s probe of a not-yet-started run, whose events file may hold an earlier incarnation's lines, leaves it false and keeps the capture rule.
`turnStartAfterSessionStart` reads the events file through the engine's `SessionSignalParser` and reports whether it holds a `SessionSignalSessionStart` and, after it in file order, a `SessionSignalTurnStart`; an engine without the parser, an absent file and an unreadable file read false.
It reads signals only, never a Claude hook name (`PATTERN-shuttle-provider-seam`).
In `checkLivenessTick`, after the tracked, live and `sessionGone` checks and the `*started` short-circuit, and before the pane capture: when `run.startupFromEvents && run.state.RecordsSessionStarts` and `turnStartAfterSessionStart` answers true, the tick sets `*started` and calls `markStarted`, and returns.
`markStarted` is the existing persist block of the `StartupReady` branch (set `run.state.Started`, save under `recordMu`, Warn on a failed save), factored out so both ready paths persist identically.
Otherwise the tick captures and classifies exactly as today, so a recognized gate is still dismissed, and a launch whose engine does not record session starts keeps the capture-only probe.
Bound: the proof is read only in `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it; a trust or bypass gate holds the first turn, so no turn start reaches the file while a gate is on screen.
This adds only a positive answer; every negative answer still goes through `classifyStartupWindow` and the output-file checks (`PATTERN-completion-signal`).

Tests, as rows of `TestStartup_NeedsSessionStartLine` (its `sessionFakeEngine` reads `SESSIONSTART` and `START`; the `"gate"` capture with a `StartupPending` script is a blank pane):
- `a session start then a turn start in the events file starts the run while every capture is pending`: `records: true`, script `StartupPending`, events `"SESSIONSTART:startup\nSTART\n"`, `wantStarted`.
- `a session start with no turn start and a pending pane stays pending and dies at the startup deadline`: `records: true`, script `StartupPending`, events `"SESSIONSTART:startup\n"`, `wantErr: "outcome \"died\""`.
- `a trust gate with a session start and no turn start is dismissed and not started`: `records: true`, script `StartupTrustPrompt`, events `"SESSIONSTART:startup\n"`, `wantEnter`, not started.
- `a launch that does not record session starts ignores a turn start in the events file`: `records: false`, script `StartupPending`, events `"SESSIONSTART:startup\nSTART\n"`, not started.
- `a turn start before the session start does not start the run`: `records: true`, script `StartupPending`, events `"START\nSESSIONSTART:startup\n"`, not started.
The started rows also assert `run.json`'s `Started` is true through `loadRunState`, adding a `wantPersisted` check to the started branch.
`TestRun_Wait_StartupWindow` gets a row proving Wait keeps the capture rule: a new `events string` field seeds the events file and, when set, runs the row over a `sessionFakeEngine`; the row `a session start then a turn start in the events file does not start Wait's probe of a booting pane` has `recordsSessionStarts: true`, script `StartupPending`, events `"SESSIONSTART:startup\nSTART\n"` and `wantOutcome: OutcomeDied`.
`doc.go`'s startup section states the events proof, its `start`-only scope and its bound.

**ImpactSummary:** Adds one Run field and one positive ready path inside checkLivenessTick, reached only from awaitStartup; Wait and Attach keep today's capture rule; shuttleengine-local.
