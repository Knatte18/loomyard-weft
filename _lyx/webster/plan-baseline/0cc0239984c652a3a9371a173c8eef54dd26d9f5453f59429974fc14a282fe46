---
format: 5
approved: true
language: go
---

# Plan: engine-model-choice

Every role's shuttle spawn spec carries the engine its model alias resolves to, through the one `shuttleengine.ModelChoice` built by `shuttleengine.NewModelChoice`, so a later `codexengine` can take a role by a model key alone.
Roles only Claude Code can run declare what they need through `Spec.Requires`, and the caller that resolved the role's key names it in the refusal.
Webster records each session's engine in its own state and reads every session through that engine, checking each role's engine before it spawns.

## Card Index

1 — spec-embeds-model-choice — Spec and OpenSpec embed ModelChoice; Result carries the run's engine; `lyx shuttle open` opens on the alias's engine
2 — seat-model-choice — seats carry a ModelChoice and every seat resolver builds it with NewModelChoice
3 — burler-model-choice — burler's per-round lists, RunOpts and the Bouncer pair are shuttle ModelChoices
4 — role-settings-model-choice — driver, arbiter, alignment and tackler settings carry a ModelChoice
5 — treadle-model-choice — treadle's profile carries judge and attempt ModelChoices
6 — round-usage-engine — round-N-usage.yaml records each half's and Bouncer pass's engine
7 — webster-roles-record-engines — webster roles resolve to ModelChoices and state.json records each session's engine
8 — webster-session-engines — webster reads each session through its recorded engine via a told lookup
9 — webster-spawn-engine-checks — run and recover-batch refuse a role engine without fork audit before any mutation
10 — merriam-instruction-files — MerriamBase counts the instruction files Master's engine names
11 — start-requirements-model-key — shuttle's Start requirements refusal becomes a typed error whose generic way forward a caller's ModelKey replaces
12 — seat-requires — seats declare their requirements and a Start refusal names the model key
13 — master-driver-requires — Master and the loom driver declare their requirements
14 — model-choice-mapping-scan — PATTERN-model-choice-mapping and its scan

## Shared Decisions

- A carrier that held `Model`, `Effort` and `Version` embeds `shuttleengine.ModelChoice`, so reads keep working through promotion and only composite literals change.
- A resolver that also resolves a timeout or spawn budget keeps it beside the choice, never inside it.
- A test that proves an alias's engine flows uses `claude`, the only engine `modelspec` names, against an engine set whose default is another name, or a `modelspec.Registry` literal naming any engine; fixtures over `shuttlefake.EngineSet` give their roles an empty engine, the set's default.
- Every refusal a bound module's verb can newly reach lands its `contracts/specs/refusal-spec.md` row on the card that adds it.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
