# Card 2 — shuttle-run-session-state

**Create:**
- `plan:internal/shuttleengine#Run.SessionState` -> `func (run *Run) SessionState() (RunSessionState, error)`

**Edit:**
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Runner.SessionState`

**Intent:**
Let a run's handle read its own session state, the reading the seat engine's chair deferral takes from each advisor (decision D10).

`Run.SessionState` returns what `Runner.SessionState` returns for the run's strand guid: the files-only fold of the run's session signals and facts at the runner's clock.
It reads files only and writes nothing, so it is safe to call from another goroutine while the run's own `Wait` runs.
It is provider-neutral: the signals come through the engine's `SessionSignalParser` capability as today, and nothing Claude-specific leaves `claudeengine` (`PATTERN-shuttle-provider-seam`).
No decision reads the wait mark through it (`PATTERN-wait-mark-display-only`).

Test: `TestRunner_SessionState` in `session_test.go` gains the assertion that the `Run` handle of the found strand answers the same reading as `Runner.SessionState` for its guid.

Name the accessor in `internal/shuttleengine/doc.go` where the session-state reading is described.

**ImpactSummary:** Adds one read-only accessor on Run; no existing caller changes.
