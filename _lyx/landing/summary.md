# Open agents in bypass mode; show and list engine, model, effort

Every agent session lyx launches into reed now starts in its engine's bypass mode without a flag, and `lyx shuttle open` lets the operator see and choose the engine, model and effort.
`lyx reed add --help` no longer shows a broken claude command line and sends agent launches to `lyx shuttle open`.

Closes #773

## What changes

- **Bypass by default.**
  The claude engine resolves an empty permission mode to bypass for an autonomous run, an interactive run and an opened session alike.
  Before, only an autonomous run did; interactive runs and `shuttle open` kept the permission dialog.
  `--permission-mode prompt` is the explicit opt-out, accepted on an interactive run or an opened session and still refused on an autonomous run; an unknown mode is still refused.
- **`shuttle open` reports its choice.**
  The success envelope adds `engine`, `model` and `effort` beside `guid`, `name` and `sessionId`.
  An empty `model` or `effort` means the provider default, since lyx does not know what the provider picks.
- **`shuttle open --list-models`.**
  It opens nothing, needs no `--role`, and prints the model registry, one row per alias sorted by alias, with `engine`, `model` and default `effort`.
  It reads the same registry `--model` resolves against.
- **`shuttle open --help`** documents both model-spec forms, `<alias>[<effort>]` and `<engine>:<model-id>[<effort>]`, points at `--list-models`, and states that bypass is the default and `prompt` the opt-out.
- **`reed add --help`** now uses a plain shell command as its example and says an agent session is opened with `lyx shuttle open --role <role>`.
  `%SID%` appears nowhere, so nothing needs substituting.
- Docs follow: `internal/shuttleengine/doc.go`, `claudeengine/doc.go`, the `OpenSpec` and `Spec` field comments, and the shuttle row of `docs/overview.md`.

## Design decisions

- The default is resolved inside the claude engine, so the bypass flag stays behind the provider seam and each engine maps the default onto its own flag.
- A Codex engine is not registered, so its equivalent (understood to be `--yolo`) lands with that engine.
- Without `--model` the provider default stays; no configured default model was added, and the envelope says so with empty values.
- The CLI still maps a resolved spec only through `shuttleengine.NewModelChoice`, and `modelspec` gains no import.

## Behavior change to know

An interactive run or opened session that passed no permission mode used to prompt and now runs in bypass.
Claude Code still applies its own settings layers' deny rules and hooks in bypass, and a run keeps every shuttle `PreToolUse` guard.
`--permission-mode prompt` restores the dialog.

## Not run by hand

- `lyx shuttle open --role r --model opus[high]` against a live tmux/reed host, to see the new `engine`, `model` and `effort` fields and the bypass flag on a real claude session.
- `lyx shuttle open --list-models` against the operator's real registry.
