# heddle: split claude-file and codex-file, add shared agents-file

A heddle entry aimed at `claude-file` rendered into both `CLAUDE.local.md` and `AGENTS.override.md`,
so there was no way to give an instruction to one agent family alone.
The file targets now separate by agent family:

- `claude-file` renders into `CLAUDE.local.md` only.
- `codex-file` renders into `AGENTS.override.md` only.
- `agents-file` is new and renders into both.

Closes #104

## Design decisions

- `agents-file` is a table row in `internal/heddleengine/target.go` with no file of its own.
  A new `Target.RendersInto` field names the file targets whose files carry its entries.
  `Render` skips such a row and `fileText` includes an entry when it names the target or a shared target that renders into it.
  The old special cases are gone: `withCodexFile` and the rule that a repository entry naming `claude-file` also reached `codex-file`.
- `heddle.json` gains a top-level `version`, written as 1.
  A store without one is read as written by the old binary: `claude-file` becomes `agents-file`,
  and a `codex-file` beside it is dropped, since `agents-file` already reaches it.
  This keeps existing entries rendering as they did.
  The upgrade runs in `Load` and `Decode`, and the next `Save` writes the version.
  A store whose version is higher than this binary's keeps it, so an older binary does not downgrade it.
- A shipped entry that names `claude-file` now reaches Claude only.
  Before, only repository entries reached `codex-file` through `claude-file`, so shipped entries did not reach it either.
- `lym heddle show agents-file` is refused with a way forward naming `claude-file` and `codex-file`, since a shared target has no text of its own.
- Each file target's files are stated in three places:
  - `lym heddle set --help` lists them.
  - `lym heddle list` prints them under a new `file_targets` field.
  - `HEDDLE.md` states them in each target's description.

## To check by hand

- The upgrade of an existing `heddle.json` against a real hub:
  run `lym heddle list` and `lym heddle set` on a hub whose store has no `version`,
  then confirm its `claude-file` entries show as `agents-file` and both agent files still render the same text.
- `internal/heddlecli/cli_integration_test.go` gained cases for the new output;
  the integration suite was not run for this description.
