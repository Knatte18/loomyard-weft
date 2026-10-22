# Agent sessions default to bypass; shuttle open shows its model choice

Every agent session lyx launches into reed now starts in its engine's bypass mode without a flag.
`lyx shuttle open` explains how to pick the engine, model and effort, says which ones the session got, and lists the available choices.
`lyx reed add --help` no longer shows a claude command line that cannot work.

## Changes

- **Bypass by default.**
  `claudeengine`'s `validatePermissionMode` resolves an empty mode to bypass (`--dangerously-skip-permissions`) for autonomous runs, interactive runs and opened sessions.
  Interactive runs and `lyx shuttle open` used to prompt by default.
  `prompt` is still the per-call opt-out on an interactive session, and is still refused on an autonomous run.
  The default is resolved inside the engine, so a future engine maps it onto its own bypass flag.
- **`lyx shuttle open`.**
  The help spells out both model-spec forms, `<alias>[<effort>]` and `<engine>:<model-id>[<effort>]`, and says what an omitted `--model` gives.
  The success envelope adds `engine`, `model` and `effort`, where an empty model or effort means the provider default.
  `--list-models` prints the loaded model registry and opens nothing, so it needs no `--role`.
  Each row is an alias with its engine, model and default effort, sorted by alias.
- **`lyx reed add`.**
  The `%SID%` example, which nothing substituted, is replaced by a plain command.
  The help now says an agent session is opened with `lyx shuttle open`.
- Docs: the `Spec`/`OpenSpec` permission-mode comments, `internal/shuttleengine/doc.go` and the shuttle row in `docs/overview.md`.

## Design decisions

- Bypass by default covers interactive runs too (`shuttle run --interactive`, a seat chair, an interactive discussion), because the operator asked for it on every agent instance in reed.
  Claude Code's own settings deny rules and hooks still apply in bypass, and every shuttle `PreToolUse` guard stays on a run.
- Without `--model`, `open` still leaves the model to the provider.
  It reports that as an empty model and effort and adds no default-model config key.
- `%SID%` is not substituted, because an agent launch no longer goes through `reed add`.
- No Codex engine is registered, so its bypass flag lands with that engine.

## Tests

- `TestValidatePermissionMode` and `TestPrepare_ThreadsSpecIntoLaunchCmds`: an empty mode on an interactive run now bypasses.
- `TestOpenLaunch`: a default open carries the bypass flag on its launch line and on its resume line.
- `TestOpenCmd_ModelAliasEngine`: checks the envelope's engine, model and effort, with a new `opus[high]` row.
- `TestOpenCmd_ListModels`: `--list-models` lists a `models.yaml` alias and the built-ins with their fields, sorted, and opens nothing.
