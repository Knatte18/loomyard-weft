# Card 6 — cross-run-readers

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#ReadAgentActivity` -> `func ReadAgentActivity(cfg Config, anchorPath string, engines EngineSet) ([]AgentActivity, error)`
- `internal/shuttleengine#ReadSessionStates` -> `func ReadSessionStates(cfg Config, anchorPath string, engines EngineSet, now time.Time) ([]RunSessionState, error)`
- `internal/shuttleengine#ReadAgentWaits` -> `func ReadAgentWaits(cfg Config, anchorPath string, engines EngineSet, now time.Time) ([]AgentWait, error)`
- `internal/shuttleengine#TestReadAgentActivity`
- `internal/shuttleengine#TestReadSessionStates`
- `internal/shuttleengine#TestReadAgentWaits`
- `internal/battencli#readAgentActivity`
- `internal/battencli#readSessionStatesByStrand` -> `func readSessionStatesByStrand(cfg shuttleengine.Config, anchor string, engines shuttleengine.EngineSet) map[string]shuttleengine.SessionState`
- `internal/battencli#readAgentStates`
- `internal/loomcli#agentActivityFor`
- `internal/loomcli#readAgentStates`
- `internal/shuttlecli#shuttleCLI`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#shuttleCLI.stateCmd`
- `internal/shuttlecli#TestStateCmd_ListsRunningRunsAndNarrowsByStrand`
- `internal/shuttlecli#TestSmokeTurnsSessionHooks`
- `internal/tendercli#shuttlePair`
- `internal/tendercli#newShuttlePair`
- `internal/tendercli#shuttlePair.Sessions`
- `internal/tendercli#shuttlePair.State`
- `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`

**Uses:**
- `internal/shuttleengines#New`
- `internal/testkit/shuttlefake#EngineSet`

**Intent:** Make the three cross-run readers dispatch each run through the engine its record names (D5, D6), and move their production callers onto the registry.

`RunState` gains `Engine string` with tag `json:"engine,omitempty"`, its doc comment saying it is the name of the engine the run was started with, and that a record without it, all written before this task, reads as the set's default engine.
Nothing writes it until card 8; the record is still rewritten through `internal/state` as today, and an older binary ignores the key (`PATTERN-tolerant-records`).

`ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits` take an `EngineSet` and, for each run they would read, resolve `engines.Lookup(rs.Engine)`.
A record naming an unregistered engine is skipped, the handling an unreadable record gets: logged through `unreadableFiles` once per episode, under a key of its own (the run directory joined with `engine`) so the record-readable recovery logged on the same pass does not end the episode, and recovered once the engine resolves again.
The bound is the one D6 states: a skipped run reads as neither busy nor waiting to the parent-lock and loop-lock decisions, as an unreadable record does today, and the state arises only when a binary lacks an engine a run was started with.
`ReadAgentActivity` takes each run's `ActivityReader` from that run's engine instead of once per call.
Each doc comment names the per-run dispatch and the skip, and `internal/shuttleengine/doc.go` gains the same in one sentence about the cross-run readers.

Production callers build the set with `shuttleengines.New(cfg)` from the shuttle config they already load, returning its error as they return a config error today:
- `battencli`'s `readAgentActivity` builds it in place of `claudeengine.NewFromConfig`, and `readSessionStatesByStrand` takes the set;
- `battencli`'s and `loomcli`'s `readAgentStates`, and `loomcli`'s `agentActivityFor`, build it in place of their `claudeengine.NewFromConfig`;
- `shuttleCLI` gains an `engines` field that `Command`'s `PersistentPreRunE` fills from `shuttleengines.New`, failing the verb with the error envelope on an error as it does a config error; `stateCmd` reads through `c.engines`.
  The `engine` field stays, set from `c.engines.Lookup("")`, only to build the runner until card 7.
- `shuttlePair` gains `engines`, which `newShuttlePair` fills from `shuttleengines.New`; `Sessions` and `State` read through it.
  Its `engine` field stays, set from the set's default, only to build the runner until card 7.
The `claudeengine` import leaves `battencli/parentlock.go`, `loomcli/looplock.go` and `tendercli/deps.go`, where nothing else uses it; `battencli/wire.go` and `loomcli/looparm.go` keep it for their runner sites until card 7.

Tests:
- `TestReadAgentActivity`, `TestReadSessionStates` and `TestReadAgentWaits` pass a set built with `NewEngineSet` around their fake engine, and each gains rows over one root holding a run of each of two registered fake engines, whose records name them, asserting each run's reading came through its own engine (each fake answers a reading the other does not); a run whose record has no `engine` key, read through the default; and a run whose record names an unregistered engine, skipped while the other runs are read, with one Warn line naming the run directory in the captured log, and read through its engine on a second call after its record is rewritten to name a registered engine, with the recovery line logged.
- `TestStateCmd_ListsRunningRunsAndNarrowsByStrand` sets `engines: shuttlefake.EngineSet(stateEngine{...})` in place of its `engine` field.
- The tmux-tier `TestSmokeTurnsSessionHooks` and `TestSmokeSeatsChairAsksAdvisor` wrap the engine they pass `ReadSessionStates` with `shuttlefake.EngineSet`.

**ImpactSummary:** Three exported readers re-signed; their callers in battencli, loomcli, shuttlecli and tendercli, two smoke tests and three reader tests follow, with no change to any reading of a single-engine hub.
