# heddle: split claude-file and codex-file, add a shared agents-file

Until now a repository entry aimed at `claude-file` rendered into both `CLAUDE.local.md` and `AGENTS.override.md`.
There was no way to give an instruction to Claude alone or Codex alone.
This change gives heddle three text file targets:

- `claude-file` renders into `CLAUDE.local.md` only.
- `codex-file` renders into `AGENTS.override.md` only.
- `agents-file` is new and shared. It has no file of its own, and its entries render into both files.

## What changed

- **Target table** (`internal/heddleengine/target.go`):
  a new `RendersInto` field marks a shared file target, and `agents-file` is its one row.
  `TargetFiles` returns the files any file target renders into.
  `fileText` takes an entry when it names the target or names a shared target that renders into it.
  This replaces the codex-file special case that took repository `claude-file` entries.
  `Render` skips the shared row, since it has no file, and the `withCodexFile` expansion of `Only` is gone.
  `Show` refuses `agents-file` and names the targets to show instead.
- **Keeping existing entries' behaviour** (`store.go`):
  heddle.json now carries a top-level `"version": 1`.
  `Load` and `Decode` read a store without a version as its writer meant it:
  each `claude-file` target becomes `agents-file`, and a `codex-file` beside it is dropped because `agents-file` already covers it.
  `MarshalJSON` always writes the version, so the next `lym heddle set`/`remove` or replay saves the migrated targets.
  The Bolt replayer decodes base, parent and commit through `Decode`.
  A commit replayed from a branch that predates this change is therefore read with the old meaning, consistently with origin.
- **Shipped layer**: unchanged.
  Its two text entries were already aimed at `claude-file` alone, and they still render into `CLAUDE.local.md` only.
  The migration runs only on heddle.json, never on `shipped.json` or the override file.
- **Saying which files each target renders to**:
  - `lym heddle list` prints `file_targets`: each file target with its files.
  - `lym heddle set --help` lists the same table, generated from the target table.
  - In HEDDLE.md, each file target's section names its files.
    `claude-file` and `codex-file` also say they carry every entry aimed at `agents-file`.
- Package docs of `heddleengine` and `heddlecli` are updated.

## Design decisions

- The migration lives in the store's own read, behind a format version, rather than in a `hubreconcile` reinterpretation.
  `configreg.Reinterpretation` rewrites a module's YAML base file, and heddle.json is not one.
  A one-time board rewrite would also leave old `claude-file` entries on branches that replay later.
  The heddle replayer would then merge those entries under the new meaning.
  Reading by version gives every read, render and replay the writer's meaning, with no step for the hub.
- `agents-file` is a row in the target table with `RendersInto`, not a special case in the renderer.
  Another shared target would be one more row.
- `Show agents-file` is refused because the shared target has no single text of its own.

## Accepted gaps

- An older binary that writes heddle.json after this one keeps the `version` key, because unknown keys are tolerated.
  A `claude-file` entry that older binary adds therefore reads as Claude-only, not as both files.
- The lym override file gains a `version` key the next time it is saved; its reads ignore it.

## Tests

- `render_test.go`:
  - `claude-file` alone no longer writes `AGENTS.override.md`.
  - An `agents-file`, `claude-file` and `codex-file` store renders each file's expected text.
  - `Only` limits the render to the named file.
  - `Show agents-file` is refused with its way forward.
- `store_test.go`: unversioned stores read `claude-file` as `agents-file` through both `Load` and `Decode`, while a versioned or saved store keeps `claude-file`.
- `overview_test.go`: the `agents-file` section, its entry and the `claude-file` reach line.
- `cli_integration_test.go`: `list`'s `file_targets` and the table in `set --help`.
