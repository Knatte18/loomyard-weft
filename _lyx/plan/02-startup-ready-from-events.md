# Card 2 — `start`'s startup probe reads readiness from the events first

**Edit:**
- `internal/shuttleengine#Run.checkLivenessTick` -> `func (run *Run) checkLivenessTick(started *bool, startupDeadline time.Time, eventsProof bool) (Outcome, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestStartup_NeedsSessionStartLine`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`

**Create:**
- `internal/shuttleengine#turnStartedAfterSessionStart`
- `internal/shuttleengine#Run.markStarted`

**Uses:**
- `internal/shuttleengine#sessionStartRecorded`
- `internal/shuttleengine#sessionFakeEngine`

**Intent:**
D1: in `start`'s probe a launch is ready once its events file holds a session-start signal and, after it, a turn-start signal, checked before the pane capture, so a collapsed or blank pane no longer holds back a provider that is past every gate and working.

`checkLivenessTick` gains `eventsProof`.
`awaitStartup` passes true and `Run.Wait` passes false, so Wait's and attach's probe of a not-yet-started run keeps today's capture rule, since that events file may hold an earlier incarnation's lines.
With `eventsProof` set, a record with `RecordsSessionStarts` and a still-false `*started` checks `turnStartedAfterSessionStart` after the reed-status, liveness and `sessionGone` checks and before `CapturePane`.
A true answer sets `*started`, persists `Started` and returns `"", nil`, exactly as a ready capture does today; `markStarted` is that persist (lock `recordMu`, set `state.Started`, `saveState`, Warn on a save failure), moved out of the `StartupReady` arm so both ready answers call it.
A false answer falls through to today's capture path, so a recognized gate is still dismissed and a ready pane still needs its session-start line.
An engine whose launch does not record session starts never reaches the check and keeps the capture-only probe.

`turnStartedAfterSessionStart` reads the events file from its start and parses it through the engine's `SessionSignalParser`, never by Claude hook name, per `PATTERN-shuttle-provider-seam`.
It answers true when a `SessionSignalTurnStart` follows a `SessionSignalSessionStart` in file order, and false for an engine without the parser, an absent or unreadable file, a turn start with no session start before it, and a session start with no turn start after it.
Bound: the proof is read only in `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks satisfy it; a trust or bypass gate holds the first turn, so no turn start reaches the file while a gate is on screen.
D1 adds only a positive answer, so every negative startup answer still goes through the `allOutputFilesExist` consultations `PATTERN-completion-signal` audits, and the audited counts in `completionsignal_enforcement_test.go` do not change.
`shuttleengine/doc.go`'s startup paragraph says a recording launch is also ready once its events file holds a session start followed by a turn start, in `start`'s probe only.

Tests, as rows of `TestStartup_NeedsSessionStartLine`, which runs `StartGated` over `sessionFakeEngine` (`SESSIONSTART:<source>` a session start, `START` a turn start) with `script` scripting `Startup`:
- "a session start then a turn start start the run while every capture is blank": `records`, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, `wantStarted`.
- The existing row "a session-start line of an earlier launch with a pane not yet ready is not started" (script `StartupPending`, a session start and no turn start) also asserts `wantErr` `outcome "died"`, the startup deadline's teardown under `shortStartupConfig`.
- "a trust gate beside a session start and no turn start is dismissed and not ready": `records`, script `StartupTrustPrompt`, events `SESSIONSTART:startup\n`, `wantEnter`, not started.
- "a turn start before any session start does not start the run": `records`, script `StartupPending`, events `START\nSESSIONSTART:startup\n`, not started.
- "a launch that does not record session starts ignores a turn start": no `records`, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n`, not started.
`TestRun_Wait_StartupWindow` gains an `events` field and a row "a session start then a turn start beside a still-booting pane on a recording launch": `recordsSessionStarts`, script `StartupPending`, events `SESSIONSTART:startup\nSTART\n` written to the run's events path, run over a `sessionFakeEngine` when `events` is set, `wantOutcome` `OutcomeDied`; it pins that Wait's probe ignores the events proof.

**ImpactSummary:** `checkLivenessTick` is re-signed with its two callers, `Run.awaitStartup` and `Run.Wait`, on this card; Wait's behaviour is unchanged and only `start`'s probe gains a positive answer, inside `shuttleengine`.

**Commit:** 2: Read start's readiness from the events before the pane
