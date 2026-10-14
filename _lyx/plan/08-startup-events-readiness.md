# Card 8 — startup-events-readiness

**Create:**
- `plan:internal/shuttleengine#turnStartAfterSessionStart` -> `func turnStartAfterSessionStart(engine Engine, eventsPath string) bool`
- `plan:internal/shuttleengine#Run.markStarted` -> `func (run *Run) markStarted(started *bool)`

**Edit:**
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, eventsProof bool) (Outcome, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine/doc.go`

**Intent:** D1: `start`'s startup probe reads readiness from the run's events ahead of the pane capture.

`checkLivenessTick` gains `eventsProof`.
`awaitStartup`, which only `start` calls, passes true; `Wait`, whose probe of a not-yet-started attached run may read an earlier incarnation's events lines, passes false and keeps the capture rule.
With `eventsProof` true and `run.state.RecordsSessionStarts` true, the tick, after its liveness and gone-session checks and before `CapturePane`, asks `turnStartAfterSessionStart`; a true answer marks the run started through `markStarted` and returns `("", nil)` without capturing.
`markStarted` is the started-and-persist block the ready-capture branch runs today (set `*started`, clear `startupHookReason`, set and save `RunState.Started` under `recordMu`, `Warn` on a save failure), and that branch calls it too, so both ready answers set and persist `Started` alike.
`turnStartAfterSessionStart` reads the whole events file and reports whether the engine's `ParseSessionSignals` yields a `SessionSignalSessionStart` followed, later in the file, by a `SessionSignalTurnStart`; an engine without `SessionSignalParser`, an absent or unreadable file and a turn start only ahead of every session start answer false.
It reads only through the engine's session-signal parser, never a Claude hook name (`PATTERN-shuttle-provider-seam`).
A false answer leaves the tick on today's capture path, so a recognized gate is still dismissed and a ready pane still needs its session-start line.
This only adds a positive answer: every negative answer still goes through the file-contract checks it goes through today (`PATTERN-completion-signal`).
Bound: the proof is read only in `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it; a trust or bypass gate holds the first turn, so no turn start arrives while one is on screen; an engine that records no session starts keeps the capture-only probe.

Tests, rows of `TestStartup_NeedsSessionStartLine` (its `sessionFakeEngine` reads `SESSIONSTART` as a session start and `START` as a turn start).
The table gains a `captures []string` field that the loop's `fakeReed` takes as its `CaptureQueue`, `[]string{"gate"}` when nil, so a row can script blank captures (`[]string{""}`); the test's doc comment adds the events-first answer to the cases it lists.
- `a session start then a turn start is ready while every capture is blank`: records, script `StartupPending`, captures `[]string{""}`, events `SESSIONSTART:startup\nSTART\n`, started.
- The existing row `a session-start line of an earlier launch with a pane not yet ready is not started` takes blank captures (`[]string{""}`) and also asserts the error holds `outcome "died"`, pinning that a session start with no turn start and blank captures dies at the startup deadline.
- `a trust gate with no turn start is dismissed and not ready`: records, script `StartupTrustPrompt` throughout, events `SESSIONSTART:startup\n`, Enter sent, not started.
- `a turn start ahead of every session start is not ready`: records, script `StartupPending`, events `START\nSESSIONSTART:startup\n`, not started.
- `a launch that does not record session starts ignores a turn start`: not recording, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, not started.
Each started row also reads `run.json` back and asserts `Started` is true.

Docs: `awaitStartup`'s and `checkLivenessTick`'s doc comments, and `internal/shuttleengine/doc.go`'s startup paragraph, say that `start`'s probe answers ready once the events file holds a session start followed by a turn start, ahead of the capture, while `Wait`'s probe keeps the capture rule.

**ImpactSummary:** `checkLivenessTick` is re-signed and its two callers, `awaitStartup` and `Wait`, both in `wait.go`, change with it; only `start`'s probe gains the events answer, and `Wait`'s behavior is unchanged.

**Commit:** 8: Answer start's startup probe from the events before the pane
