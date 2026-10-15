# Card 7 — Webster roles resolve to ModelChoices and its state records each session's engine

**Edit:**
- `internal/websterengine#ResolveRoles` -> `func ResolveRoles(cfg Config, reg modelspec.Registry) (map[Role]shuttleengine.ModelChoice, error)`
- `internal/websterengine#RunDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#Run`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#State`
- `internal/websterengine#BatchState`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/loomcli#loomCLI.wire`
- `internal/websterengine_test#TestResolveRoles`
- `internal/websterengine_test#newRunFixtureOver`
- `internal/websterengine_test#newRecoverFixtureOver`
- `internal/websterengine_test#TestRun_MasterSpawn`
- `internal/websterengine_test#seedShuttleRunState`
- `internal/websterengine_test#TestBeginBatch_Record`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine_test#TestState_RoundTrip`
- `internal/webstercli#newVerbsFixture`

**Intent:**
D2 for webster's two roles and D6's recording half.
`ResolveRoles` returns `map[Role]shuttleengine.ModelChoice`, each built by `shuttleengine.NewModelChoice`; `RunDeps.Roles`, `RecoverDeps.Roles` and `websterCLI.roles` take that type, and `Run` and `recoverSpawn` set `ModelChoice: choice` from the role's entry, so Master and the recovery strand spawn on their aliases' engines.

Webster records each session's engine beside the session id it records, because a shuttle run record does not outlive its run:
- `State` gains `MasterEngine string` (json `masterEngine`, omitempty) beside `MasterSessionID`; `Run` sets it from the `RunState.Engine` its `FindRun` of Master's strand returns, in the same save that records `MasterSessionID`.
- `BatchState` gains `Engine string` (json `engine`, omitempty) beside `SessionID` and `RecoveryEngines map[string]string` (json `recoveryEngines`, omitempty) beside `RecoverySessions`.
- `BeginBatch` copies `State.MasterEngine` into the batch's `Engine` as it copies `MasterSessionID` into `SessionID`, and its re-begin record carries the prior record's `RecoveryEngines` forward as it carries `RecoverySessions`.
- `recoverSpawn` writes the spawned run's `RunState.Engine` into `RecoveryEngines` under the session id it appends to `RecoverySessions`, starting from a copy of the prior record's map, so an entry is dropped only when its session id is.
Both keys are additive, and a state written before this change reads with them absent (`PATTERN-tolerant-records`).
`doc.go`'s state section names the three keys, and `roles.go`'s file header, which says the roles map onto resolved model-spec values, is reworded to say they map onto model choices.

Tests:
- `TestResolveRoles` asserts each role's `Engine` is its alias's engine.
- `seedShuttleRunState` writes `Engine: shuttlefake.EngineName` into the run record it seeds, as a real `Start` would; its other callers assert nothing about the engine.
- `newRunFixtureOver`, `newRecoverFixtureOver` and `newVerbsFixture` build their roles as `shuttleengine.ModelChoice` values with an empty `Engine`, the default of their `shuttlefake.EngineSet`, since a role naming `claude` would now be refused by a set that registers only `fake`.
- `TestRun_MasterSpawn` asserts the spawned spec's `Engine` is the master choice's and that `state.json` records `masterEngine` `fake`, the engine the seeded Master run record names.
- `TestBeginBatch_Record` asserts a begun batch records `engine` equal to `masterEngine` and that a re-begin over a record with `recoveryEngines` keeps them.
- `TestRecoverSpawnOrAttach` asserts a first recovery records its session's engine under its session id, and a respawn keeps the first entry and adds the second.
- `TestState_RoundTrip` round-trips the three keys and reads a state written without them.

**ImpactSummary:** webster's role map changes type across websterengine, webstercli and loomcli; state.json gains three additive keys written at Master spawn, begin-batch and recovery spawn.

**Commit:** 7: webster roles resolve to ModelChoices and state records each session's engine
