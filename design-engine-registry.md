# Engine registry: engine per run, one constructor, narrower Engine interface

`engine-registry` · Task · enhancement, shuttle, config, loom, webster, burler, orch, batten · running · Merge-Burler-Fix

## Goal

First of the tasks that make lyx ready for a second LLM engine (Codex CLI) before Codex is installed; the series is `engine-registry`, `engine-config-seams`, `engine-model-choice`, `engine-usage-readers`, and creel in three stages (`creel-core`, `creel-notices`, `creel-agents`), from a planning agent's report of 2026-10-15 (not committed).
This task gives shuttle an engine registry and records the engine per run, so a later `codexengine` plugs in at one place.

## Operator decision (2026-10-15)

Build the tasks of the Codex-readiness plan, in order; everything that needs Codex installed stays in note `codex-engine`.
Creel is a prerequisite for Codex: agents reach their parent and each other through creel, not Claude Code's `SendMessage`, so the plan's interim `neutral-parent-asks` (hold-based asks plus `lyx shuttle send`) is dropped.
Operator view, not a decision: that the model spec's engine is dropped today is natural while lyx has one engine.

## Seen (verified 2026-10-15)

- `claudeengine.NewFromConfig`/`New` is hardwired at 17 sites in 11 files: `burlercli/wiring.go`, `webstercli/wiring.go`, `loomcli/{wiring,looparm,looplock}.go`, `orchcli/cli.go`, `battencli/{wire,parentlock,drive}.go`, `tendercli/deps.go`, `shuttlecli/cli.go`, `cliwire/sessionnamer.go`; `orchcli/resumecontext.go` calls `claudeengine.SessionStartContext`.
- `modelspec`'s `knownEngines` is `{claude}`; `shuttleengine.Spec` has no `Engine` field and the run record (`rundir.go`) stores none.
- The cross-run readers (`ReadAgentActivity`, `ReadSessionStates`, `ReadAgentWaits`) and `Runner` take a single engine.
- The `Engine` interface makes every engine implement `AuditForks*`, `ModelSwitchSequence` and `ColorSequence`, which are Claude, orch or Webster concerns.

## Scope

- `Spec.Engine`, and a neutral `engine: claude` default key in `shuttle.yaml` for specs with no model spec (the orch, a bare `lyx shuttle run`).
- An engine set (name to Engine); an unknown name is refused, naming the registered engines.
- One `Runner` over the set: `Start` picks by `Spec.Engine`; attach, send, interrupt, inject, stop, resume, open and the sweep pick from the run record.
- The run record gains `engine`; an absent value reads as the default engine.
- The cross-run readers dispatch per run.
- One registry constructor replaces the 17 sites; the orch's resume context goes through an engine capability.
- `AuditForks*` and `ModelSwitchSequence` become optional capabilities.
- A neutral `Spec.Requires` capability list (forks, peer messaging, background wake) checked at `Start`, refusing with a way forward.
- An `InstructionFiles()` capability (Claude: `CLAUDE.md`, `CLAUDE.local.md`).
- A shared `shuttleengine.ModelChoice{Engine, Model, Effort, Version}` built from `modelspec.Resolved`, defined here and adopted by `engine-model-choice`.
- A PATTERN entry: only the registry constructs or imports a concrete engine, and provider specifics live only under `shuttleengine/<engine>`; the seam test checks it.

## Out

Config key moves and guardrails (`engine-config-seams`), messaging (creel), the role model carriers (`engine-model-choice`), tokencount (`engine-usage-readers`), anything Codex.

## Design points: the planning agent's recommendations, not yet decided by the operator

- The registry lives in `cliwire`, which already holds the sole wiring (alternative: a new `internal/shuttleengines`).

## Operator decision (2026-10-15): the engine comes from the model alias

The model aliases were designed for this: `sonnet[high]` is an alias for engine `claude`, model `sonnet`, effort `high`.
Codex gets the same form for its models (Luna, Sol, Astra): `sol[medium]` is an alias for engine `codex`, model `sol`, effort `medium`.
So a role's engine follows from its model spec alias in `models.yaml`, the existing contract in `llm-model-spec.md`; there is no separate `engine:` key per module.

## Further design points (recommendations, continued)
- One Runner over the engine set (alternative: one per engine, which leaves send, interrupt, tender and the readers guessing).
- Claude-only roles are gated by `Spec.Requires` at `Start`, since config cannot see the registry.
- `modelspec` keeps a closed engine-name set, pinned equal to the registry by a test; `codex` is added only with the engine.
