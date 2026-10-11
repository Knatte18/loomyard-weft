# Six small fixes: darn prompt, batten window, reconcile, --apply, notes

Six independent fixes, each small on its own.

Closes #531

## What changes

- **Darn and landing prompts.**
  Both stencils read the task with `lyx board get '{"slug":"{{.slug}}"}'`; the bare slug failed with `invalid json`.
  The final-summary spec is renamed `contracts/specs/loom-final-summary-spec.md`, embedded and registered in `contracts/specs`, so it deploys to `<specs_dir>/loom/loom-final-summary-spec.md`.
  The darn stencil names that deployed path instead of a repo path that exists only in loomyard's own repository.
  Every reference to the old name moved.
- **`docs/overview.md`.**
  The batten paragraph says a board entry's recipe is `loom`, `darn` or empty, matching the paragraph's opening line and the code.
- **Batten terminal window.**
  `InnerRun` no longer clears its terminal-opened marker when the child has no status file, so a child respawned within one batten run opens no second window.
  `internal/battenshed` exports one function that clears the marker, and batten's pre-run and pre-step hooks call it where they seed a fresh batten status.
- **`lyx config reconcile`.**
  The `shuttle`, `orch` and `loom` modules register `Migrate` hooks in `internal/configreg` that raise `poll_interval_ms` (shuttle, orch) and `step_idle_timeout_min` (loom) to their loader's floor.
  Each rewrite is reported in `Migrated`, on a dry run and an applied run alike.
  The hooks read the engines' existing floor constants and edit the YAML nodes through a new helper in `internal/yamlengine`, so other keys and comments survive.
  A null value is raised and reported with old value `null`.
  An `${env:...}` marker or a non-integer value is left alone with no report line.
- **Destructive verbs.**
  `lyx fabric unwire`, `lyx board retire-legacy` and `lyx board prune` change nothing without `--apply`.
  The dry run exits 0 and prints the applied envelope filled with what it would change, plus `"applied": false`.
  `retire-legacy` also gains a `legacy_files` field in both modes, and refuses on a missing precondition with or without `--apply`.
  `plugins/ly/skills/board/SKILL.md`, the board README text in `internal/boardengine` and the `tools/sandbox` suites pass `--apply`.
- **Command index.**
  `internal/clihelp` adds the `lyx.index-note-audience` annotation, and an index note prints only under the audience it names.
  The `board` group names `operator`, so the payload legend no longer appears in the internal index.
  A test refuses a command with an index note and no valid note audience.
- **`WriteCarryOver`.**
  It re-reads the decision record before its rename and refuses, writing nothing, when the bytes differ from those it read at its start.
  After the rename it returns an error when the on-disk bytes differ from what it wrote.
  `contracts/specs/refusal-spec.md` gains a transient row whose way forward is re-running `lyx loom start`.

## Design decisions

- The reconcile fix covers values below a floor, not values equal to an old template default: reconcile keeps no record of earlier templates.
  A hook only raises what its own loader already raises at every load, so the effective configuration never changes.
- Floors that do not come from a config module (batten's `poll_interval_s`, shed's loop idle window) are out of scope.
- Bare `fabric unwire` stays out of `fabricengine.FabricReaderVerbs`, so the fabric role guard still refuses it from a sandboxed role.
- `fabric unwire`'s dry run records no mutation, keeping the `MutationRecord` envelope.
- The new flag is `--apply`, matching `lyx fabric prune` and `lyx fabric cleanup`.
- `orchengine`'s import allowlist in `seam_enforcement_test.go` gains `yamlengine`.

## Known limits to check

- `WriteCarryOver` holds no lock shared with `lyx loom decision add`.
  A write landing between the pre-rename compare and the rename is overwritten with no error, and a read-modify-write that reads before the rename and writes after the post-rename re-read drops the carry-over entry silently.
- The terminal marker is cleared only where batten seeds a fresh status.
  If the batten status vanishes mid-run while the scratch marker survives, the next `lyx batten run` or `step` clears the marker, and a spawn then opens a second window beside the first.
- A hub with an operator-edited copy of the darn stencil keeps the old prompt: `stencilstore` never overwrites a hash-mismatched file.
- The spec rename changes the spec names in `lyx stencil list`.
- The reconcile rule for non-integer values rests on the loader rejecting them at load, as it does today.
  Check that premise for `abc`-style values in the three modules.

## Check by hand

- Run `lyx config reconcile` and then `--apply` in a worktree whose `shuttle.yaml` holds `poll_interval_ms: 500`, and confirm one `Migrated` line and a file written at the floor.
- Run bare `lyx fabric unwire`, `lyx board prune` and `lyx board retire-legacy` in a scratch pair and confirm `"applied": false` and no change, then repeat with `--apply`.
- Run `lyx batten run <slug> --window` and confirm one window across a child respawn.
- Compare `lyx` command index output for the internal and operator audiences on the `board` line.
