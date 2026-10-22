# Decision record — shuttle-open-agent-defaults

## Goal

Every agent session lyx launches into reed starts in its engine's bypass mode with no flag, `lyx shuttle open` included.
`lyx shuttle open` lets the operator see and choose the engine, model and effort it opens, and says which it got.
`lyx reed add --help` no longer shows a broken claude command line and sends an agent launch to `lyx shuttle open`.

## Scope

In:

- The claude engine's permission-mode resolution, for runs of either mode and for opened sessions.
- `lyx shuttle open`: its help text, its success envelope and a flag that lists the model registry.
- `lyx reed add`'s help example.
- The docs these change: `internal/shuttleengine/doc.go`, the `OpenSpec` and `Spec` field comments, and `docs/overview.md`'s shuttle row.

Out:

- A Codex engine: none is registered; its bypass equivalent lands with it, in its own engine.
- Substituting `%SID%` in `reed add`: the help stops naming it, so nothing needs substituting.
- A configured default model for `shuttle open`: without `--model` the provider default stays, and the envelope says so.

## Decisions

### D1 — Bypass is the engine default for every reed session

The claude engine resolves an empty permission mode to bypass for an autonomous run, an interactive run and an opened session alike.
`prompt` stays the explicit opt-out, accepted on an interactive run or an opened session and refused on an autonomous run as today.
The resolution stays inside the engine, so each engine maps the default onto its own bypass flag.
Bound: this drops the permission dialog of interactive runs and opened sessions that pass no mode.
Claude Code still applies its own settings layers' deny rules and hooks in bypass, a run keeps every shuttle `PreToolUse` guard, and `--permission-mode prompt` restores the dialog per call.

### D2 — `shuttle open` help documents the model-spec grammar

The help names both spec forms, `<alias>[<effort>]` and `<engine>:<model-id>[<effort>]`, points at `--list-models` for the aliases, and says bypass is the default and `prompt` the opt-out.

### D3 — `shuttle open` reports the engine, model and effort it opened

The success envelope adds `engine`, `model` and `effort` beside `guid`, `name` and `sessionId`.
`engine` is the resolved engine name; `model` and `effort` are the resolved choice, empty meaning the provider default.

### D4 — `shuttle open --list-models` lists the registry

With `--list-models`, `open` opens nothing and needs no `--role`: it prints the loaded model registry, one row per alias sorted by alias, with its engine, model and default effort.
Rationale: the operator picks from the same registry `--model` resolves against, with no second source.

### D5 — `reed add` help points agent launches at `shuttle open`

The example runs a plain shell command, and the help states that an agent session is opened with `lyx shuttle open`, not `reed add`.

## Constraints

- `PATTERN-shuttle-provider-seam`: the bypass default and its flag stay in `claudeengine`.
- `PATTERN-model-choice-mapping`: the CLI maps a resolved spec only through `shuttleengine.NewModelChoice`.
- `PATTERN-leaf-packages`: `modelspec` gains no import.
- `PATTERN-comment-line-breaks`, and docs land in the same commit as the code.

## Auto-mode assumptions

- "Every agent instance in reed" covers interactive runs (`shuttle run --interactive`, a seat chair, an interactive discussion), not only `shuttle open`.
- An empty model or effort in the envelope is the honest report of a provider default; lyx does not know which model the provider picks.

## Open risks

- An operator who relied on an interactive run prompting by default now gets bypass silently; the help and docs name `prompt` as the way back.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - claudeengine-doc-open-bypass-stale (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- An opened session and an interactive run with no permission mode carry the engine's bypass flag on both launch and resume lines.
- `prompt` on an opened session or an interactive run carries no bypass flag; `prompt` on an autonomous run and an unknown mode are still refused.
- `lyx shuttle open --role r --model opus[high]` reports `engine` `claude`, `model` `opus` and `effort` `high`; with no `--model` it reports the set's default engine and empty model and effort.
- `lyx shuttle open --list-models` with no `--role` lists every registry alias with its engine, model and default effort, and opens nothing.
- `lyx reed add --help` names no `%SID%` and names `lyx shuttle open`.
