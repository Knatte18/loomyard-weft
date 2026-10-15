# Engine registry: shuttle picks and records each run's engine

Shuttle now picks each run's engine from a registry of named engines and records the name in the run record.
A later `codexengine` plugs in with one registry entry and one `modelspec` engine name, with no change at any shuttle caller.
Nothing here needs Codex installed, and `claude` stays the only registered engine.

## What changes

- `Spec.Engine` and `OpenSpec.Engine` name the engine; empty means the default.
  A new `engine:` key in `shuttle.yaml` (template value `claude`) sets that default.
  An empty value is a config error at load.
- `shuttleengine.EngineSet` maps engine name to `Engine` plus the default name.
  A lookup of an unknown name is refused, naming the registered engines.
- New package `internal/shuttleengines` holds the registry and `New(cfg)`, and is the only production importer of a concrete engine.
  `New` refuses a `Config.Engine` that names no registered engine, and an engine that declares `forks` without the fork-audit capability.
  `SessionNamer()` moved here from `cliwire`, and `cliwire/sessionnamer.go` is deleted.
- Every production construction site of `claudeengine` now goes through the registry.
- One `Runner` over the set.
  `Start` and `Open` resolve the spec's engine, refuse an unknown name before any pane exists, and write the name into `RunState.Engine`.
  Every other engine-consulting operation resolves the engine from the run's record; an absent value reads as the default.
- `ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits` take the set and dispatch per run record.
  A record naming an unregistered engine is skipped and logged.
- `Engine` shrinks.
  `AuditForks*` move to an optional fork-audit capability, and `websterengine` takes that capability.
  `ModelSwitchSequence` is deleted from the interface, every implementation and fake, and the `submit_settle_ms` template comment.
  No production code called it, and no board entry plans a live model switch.
- `Spec.Requires` is a closed list (`forks`, `peer-messaging`, `background-wake`) checked at `Start` and attach, against an optional `Supports(Requirement)` capability.
  `ForkSubagents` implies `forks`.
- New optional capabilities: `InstructionFiles()` (Claude answers `CLAUDE.md`, `CLAUDE.local.md`) and resume-context rendering, which `orchcli`'s `resume-context` verb now uses instead of `claudeengine.SessionStartContext`.
- `shuttleengine.ModelChoice{Engine, Model, Effort, Version}` with a constructor from `modelspec.Resolved`; no caller adopts it yet.
- `modelspec` exports its closed engine-name set, and a test in `internal/shuttleengines` pins it equal to the registry's names.
- `PATTERN-shuttle-provider-seam` is amended, and its scan now fails on a production import of a concrete engine package from outside its own directory and `internal/shuttleengines`.
  `internal/shuttleengines` joins `PATTERN-told-geometry`'s bound packages.
  Docs: the `doc.go` files of `shuttleengines`, `shuttleengine` and `cliwire`, and `docs/overview.md`.

## Design decisions to know

- `InterruptSettle` and the guid-less context-token, compaction and skill-load reads gain the guid of the run they read, so the engine resolves from that run's record.
  An unresolvable `InterruptSettle` returns an error; tender and the orch watcher log it and wait the longest settle they did resolve.
- Stop, the step-stop sweep and strand removal resolve no engine, so a record naming an unregistered engine never blocks a stop or teardown.
- On attach, the record's engine wins over the caller's spec, and a mismatch is logged.
  An attach whose recorded engine does not meet the `Requires` (joined with `forks` for a `ForkSubagents` record) is refused with its own way forward: `lyx reed remove <strand guid>`, then re-step once the record is older than twice `startup_timeout_s`.
  A finished run is harvested without the check.
- The `Requires` refusals are shuttle errors with no row in `contracts/specs/refusal-spec.md`, because they are unreachable while Claude is the only engine and declares all three requirements.
  The tasks that make them reachable (`engine-model-choice`, `codex-engine`) land the rows.
- `websterengine` audits and parses events through the default engine, which equals the batch's engine only while one engine is registered.
  `webstercli` and `loomcli` refuse a default engine without fork audit, with a row in `contracts/specs/refusal-spec.md`.
- Run records written before this change have no `engine` key and read as the default configured at read time.
  Long-lived holders (tender, orch watcher, batten's drive) keep the default they started with until restart.

## To check by hand

- The `tmux` and `llm` test tiers were compile-checked only and not run.
- `creel-core`, planned in parallel, calls shuttle's idle check and send, whose `Runner` constructors and guid-less methods changed here; whichever lands second adapts.
- The `PATTERN.md` entry for the seam is shorter than the background file, to fit `lint-pattern`'s headroom; check it still states both halves of the rule.
- `resumeContextCmd` resolves the capability before calling `ResumeContext`; check the failure path leaves no delivery mark.
