# Card 7 — Startup needs a SessionStart line beside the pane's ready reading

**Create:**
- `plan:internal/shuttleengine#sessionStartRecorded` -> `func sessionStartRecorded(engine Engine, eventsPath string) (bool, error)`

**Edit:**
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
D4.
Built only when card 1's corpus shows the `--settings` file's `SessionStart(startup)` line landing on its own with no turn typed; under card 1's blocker this card is skipped and reported.

`Launch` gains `RecordsSessionStarts bool`, the engine's provider-neutral declaration that its launch records a session-start line; `Claude.Prepare` sets it true, which card 1's settings guarantee.
`RunState` gains `RecordsSessionStarts bool` (`json:"recordsSessionStarts,omitempty"`), which `Runner.start` copies from the launch, so an attached run reads it from its record;
a record an older binary wrote decodes false and keeps today's pane-only startup, as does every engine that leaves the field false.

`sessionStartRecorded` reports whether the events file holds a session-start signal of any source, from its start; it returns the read error for an unreadable file and false with no error for an absent one, and an engine without `SessionSignalParser` reads false.
In `Run.checkLivenessTick`, for a record with `RecordsSessionStarts`, a `StartupReady` pane marks the run started only when `sessionStartRecorded` also finds a line.
Otherwise the tick stays not started, records on `Run` the reason (no `SessionStart` line in the events file, naming its path, or the read error) and returns through `classifyStartupWindow` as a still-booting pane does.
Because the rule lives in `checkLivenessTick`, `Run.awaitStartup` and an attached run's `Run.Wait` that is not yet started apply it alike.
The gate detection and dismissal run exactly as today until both hold, whatever the line says.
`Run.abandonStartup` appends the recorded reason to its error when `cause` is nil and the run never started, before the pane's last lines.
Bound: a `SessionStart` line left by an earlier launch of the same run satisfies the hook half, and the pane half still gates it.

`startup_test.go` gains the Startup criterion, extending `TestStartup_TrustPromptThenReady` and `TestStartup_GateNeverClearsUntilWindowExpires` where their tables fit:
a ready pane with no `SessionStart` line is not started and times out naming the missing line and the events path;
an unreadable events file times out naming that;
a `SessionStart` line with a gate on screen still dismisses the gate and waits for the ready pane;
an earlier launch's `SessionStart` line with a pane not yet ready is not started;
an engine whose launch leaves `RecordsSessionStarts` false starts on the pane alone.
`TestRun_Wait_StartupWindow` gains an attached, not-yet-started run applying the same rule.
`TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` asserts the launch's `RecordsSessionStarts`.
Both `doc.go` files' startup paragraphs state the rule.

**ImpactSummary:** Claude runs now need the hook line beside the ready pane to start, so a gate caret the classifier misses no longer reads as ready; engines and older records without the flag keep pane-only startup, and the startup error names a missing or unreadable line.
