# lyx config --set refuses a value the module's loader rejects

`lyx config <module> --set` now checks every write against the module's own loader and refuses a value the loader would reject.
Before, `lyx config loom --layer local --set 'plan=["opus[medium]", "sonnet[high]"]'` reported `ok: true` and stored a quoted string in `plan`, a single-model key, so the next reader failed with a modelspec error.
Now the write is refused, the error names the module and the key, and the layer file is left as it was (or still absent).
A valid per-round list on a per-round key, such as `plan_review`, is still written as a list.

Closes #620

## Design decisions

- `configreg.Module` gains `CheckLoad`, which runs the module's loader over a `configengine.Dirs` stack and returns only its error.
  Every row in `Modules()` sets it, through the `checkLoad` and `checkLoadNamed` adapters for the loaders' two signatures.
  `configreg` already imports every module package, so `--set` checks exactly what a later reader checks.
- The check runs after the write, because the loaders read the stack from disk.
  `setChecked` in `configcli` snapshots the layer file, runs `configengine.Set`, then the load check, and restores the snapshot on refusal.
  It covers the worktree and local layers; `setHubWide` reuses `newLoadFailure` for the board-dir write of a hub-wide module or `--layer default`.
  The hub-wide path keeps its strict `validateHubFile` check, then loads the board dir alone, and nothing is committed on refusal.
- Only a new load failure refuses.
  The check also runs before the write, and the write is refused only when the after-write error text differs from the before-write one.
  A broken lower layer or an uninitialised base layer therefore cannot block `--set` from writing or repairing a layer.
  The bound: a stack already failing for any reason lets a write through that leaves that same failure in place, and a new bad value masked by an earlier failure the loader reports first also gets through.
- A loader that reads another module's file, such as loom reading `burler.yaml` and `models.yaml`, makes a write depend on that file too; the before/after rule keeps an already-broken neighbour from blocking it.
- Out of scope: the interactive editor paths (`lyx config <module>` without `--set`, and `lyx config menu`), reconcile, and the shape rules of `internal/yamlengine`.

`internal/configcli/doc.go`, `lyx config --help` and `docs/overview.md` describe the new refusal.
The `--help` text also drops "never dropped" and "actually" from the preserved-keys sentence.
New tests extend the existing `configcli` set tests, and `configreg_test.go` asserts every module sets `CheckLoad`.

## To check by hand

Nothing outstanding beyond a run of `lyx gate test ./...` and `lyx gate test --tags integration ./...`.
