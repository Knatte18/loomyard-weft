# Card 8 — state-reader

**Create:**
- `internal/shuttleengine/sessionread.go`
- `internal/shuttleengine/sessionread_test.go`
- `plan:internal/shuttleengine#ReadSessionStates` -> `func ReadSessionStates(cfg Config, anchorPath string, engine Engine, now time.Time) ([]RunSessionState, error)`
- `plan:internal/shuttleengine#RunSessionState` -> `type RunSessionState struct`
- `plan:internal/shuttleengine#Runner.SessionState` -> `func (r *Runner) SessionState(guid string) (RunSessionState, error)`

**Edit:**
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#SessionState`
- `internal/shuttleengine#SessionFacts`
- `internal/shuttleengine#SessionSignalParser`
- `plan:internal/shuttleengine#SessionProber`
- `internal/shuttleengine#ReadAgentActivity`
- `internal/shuttleengine#runDirRoot`
- `internal/shuttleengine#loadRunState`
- `internal/shuttleengine#ActivityReader`

**Intent:**
Add the files-only reader of a run's session state (D3's facts, D6's selection), which `lyx shuttle state`, batten and the orch watcher call in later cards.

`RunSessionState` carries the strand name (the guid for a record without one), the strand guid, the current `SessionState` and its history.
`ReadSessionStates` returns one per run directory under the run-directory root of `anchorPath` whose record reads `running`, whether or not its waiting process is alive, so a run whose Wait died still shows; non-run directories, missing and undecodable records are skipped as `ReadAgentActivity` skips them, an undecodable one logged.
`Runner.SessionState(guid)` reads the one run `FindRun` finds for guid, with the runner's clock; the orch watcher reaches the orch session's record this way.

For one run, in `sessionread.go`, one helper reads the facts and folds a zero `SessionFold` once:
- The signals past `run.json`'s `promptOffset`, through the engine's `SessionSignalParser`; an unreadable events file sets `EventsUnreadable`; an engine without the parser reads `unknown` with cause `unsupported`.
- `Interactive` from the record; `OutputsExist` from one stat of the record's `outputFiles`, taken once per call.
- The session id: the `SessionID` of the newest signal that carries one, else the record's `sessionId`.
- `Liveness` from the engine's `SessionProber` for that id, unproven without the capability.
- The transcript API-error marker of the newest turn end, from the engine's `ActivityReader` over that turn end's `Raw`; the interrupt marker of the newest turn start, from the `SessionProber`.
- `ReadAt` is the `now` the caller passes, so tests inject the time.
The fact-reading helper is shaped so card 13's wait loop reuses its liveness and transcript reads over its own incremental signals.
The reader writes nothing.

Tests, untagged, in one table-driven top-level function in `sessionread_test.go`, over run directories the test writes and an engine double answering parser, prober and activity calls: a running record with a stamped turn end and outputs present reads `idle-done` stamped with the turn end's time; one whose waiting process pid is dead still reads; a non-running record, a non-run directory and an undecodable record are skipped; signals before the prompt offset are not folded; the liveness lookup uses the newest signal's session id over the record's; a missing parser reads `unknown` `unsupported`; an absent root returns nothing and no error.
`session_test.go` gains one top-level function for `Runner.SessionState` over a found and an unknown guid, since no existing `Runner` test covers a state read.
`cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine` row rises by 2, for those two functions.

Docs: `shuttleengine/doc.go` gains a section on the session-state model in shadow mode: the signals, the states and their precedence, the facts and their sources, and that no consumer acts on it.
The section states the handover of the added decision "Output files first covers output-derived states only": `dead` and `unknown` outrank `idle-done`, so a finished run can read `dead`, and the first consumer that acts on `dead` applies `PATTERN-completion-signal`'s output check before acting.

**ImpactSummary:** Adds read-only session-state readers to shuttleengine and a Runner method; no existing caller changes.
