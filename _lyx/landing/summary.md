# Every role spec carries its resolved engine through ModelChoice

Role spawn specs now take engine, model and effort from the model alias through the one `shuttleengine.ModelChoice`, so a later `codexengine` can take a role by a model key alone.
Before, about 17 sites copied `Model`, `Params["effort"]` and `Params["version"]` from a `modelspec.Resolved` by hand and dropped the engine; no consumer read `Resolved.Engine`.

## What changes

- `shuttleengine.Spec` and `OpenSpec` embed `ModelChoice` in place of their own `Model`, `Effort`, `Version` and `Engine` fields.
  Reads such as `spec.Model` keep working through promotion; a construction sets `ModelChoice: choice`.
  An empty `ModelChoice.Engine` is still the engine set's default.
- `shuttleengine.NewModelChoice` is the one mapping from a `Resolved`.
  The new `PATTERN-model-choice-mapping` and its scan test in `internal/shuttleengine` fail any production read of `Resolved.Engine`, `Model` or `Params` outside `internal/modelspec` and `NewModelChoice`.
- The separate carriers adopt `ModelChoice`:
  `burlerengine.ModelChoice` is deleted, and `RoundModels`, `RunOpts` and `shedadapters.BouncerModels` use the shuttle type, so each round of a per-round list runs on its own entry's engine.
  Treadle's `Profile` holds a judge and an attempt `ModelChoice`, and `seatengine.Seat` and `alignment.Settings` hold one.
  `DriverSettings` is deleted.
  The `lyx burler run` flags `--review-model`, `--review-effort`, `--fix-model` and `--fix-effort` still take a provider model string and build a choice with an empty engine.
- `shuttleengine.Result` gains `Engine`, the engine name the run record holds.
  Each half and Bouncer pass in `round-N-usage.yaml` gains an `engine` key from it, omitted when there is no run record.
- Claude-only roles declare `Spec.Requires`: Webster Master and the loom driver declare `background-wake`, and seats with advisors declare `peer-messaging`; the roles whose stencils fork rely on `ForkSubagents`, which implies `forks`.
  A model key whose engine cannot meet a requirement is refused at `Start`, before any pane exists, and the refusal names the key.
  The refusals have rows in the webster and loom sections of `contracts/specs/refusal-spec.md`.
- Webster audits forks and parses events through each session's recorded engine instead of the default engine.
  `State` gains `masterEngine`, each batch gains `engine` and `recoveryEngines`, all additive.
  `run` and `recover-batch` refuse a role engine without fork audit before any mutation, naming `master` or `recovery`.
- `MerriamBase` counts the instruction files Master's engine names, handed in through `Deps`, instead of `CLAUDE.md` and `CLAUDE.local.md`.

## Design decisions to know

- The spawn checks sit in the engine functions, not the wiring, because the wiring runs for every verb.
  `run` checks them ahead of `AcquireStateMutation`, so a wrong key fails before Master spends a turn.
- Webster stores each session's engine in its own state because a shuttle run record does not outlive the run.
  A state entry written before this change has no engine and reads as the current default.
  If `engine:` in `shuttle.yaml` then names an engine that differs from the one that ran the session, the read ends in that engine's audit error or returns an empty audit, which can pass an audit or hide a run's own path.
  This is accepted while one engine is registered and revisited when `codex-engine` lands.
- Until `creel-agents` lands, a role with a parent asks it through `SendMessage`, so a non-Claude engine on such a role would lose those asks with no refusal.
  `creel-agents` also revisits the `peer-messaging` declarations on seats.
- `creel-core`, `engine-config-seams` and `shuttle-open-permission` also touch `shuttleengine` or `OpenSpec`; whichever lands second adapts its `Spec` literals.

## Check by hand

- A webster run from a state written before this change, with `engine:` in `shuttle.yaml` set to a different engine, was not exercised.
- `lyx burler run` with a cluster-fan profile names the default engine in its `Requires` refusal and has no refusal-spec row; its message was not checked by hand.
