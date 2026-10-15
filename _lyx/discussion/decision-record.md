# Decision record — lyx config --set refuses a value the module's loader rejects

## Goal

`lyx config <module> --set` refuses a value that the module's own loader would reject, names the key, and leaves the layer file as it was.
The case from issue #620: `lyx config loom --layer local --set 'plan=["opus[medium]", "sonnet[high]"]'` today writes a quoted string to the `plan` key, which takes one model-spec, and reports success.

## Scope

- Every `--set` write: the worktree layer, the local layer, and the board-dir write of a hub-wide module or of `--layer default`.
- Every registered config module gets a load check that runs its own loader.
- Out of scope: the interactive editor paths (`lyx config <module>` without `--set`, and `lyx config menu`), reconcile, and the shape rules of `internal/yamlengine`.

## Decisions

- **The registry carries a load check per module.**
  `configreg.Module` gains a field holding a function that runs the module's own loader over a `configengine.Dirs` stack and returns only its error.
  Every row in `Modules()` sets it, through small adapters over the loaders' two signatures.
  Rationale: configreg already imports every module package, and the loader is the one place that knows which values a module takes, so `--set` checks exactly what a later reader checks.
- **The check runs after the write, and a refused write is rolled back.**
  `configcli` snapshots the layer file, runs `configengine.Set`, then the load check, and restores the snapshot when the check refuses.
  The refusal names the module, carries the loader's error, which names the key, and says the file is unchanged.
  Rationale: the loader reads the stack from disk, so the written file is what it must see; the snapshot and rollback already exist for the hub-wide path.
- **Only a new load failure refuses.**
  The check also runs before the write; the write is refused only when the after-write load fails with an error text the before-write load did not produce.
  Bound: a stack already failing to load for any reason lets a write through that leaves that same failure in place, and a new bad value masked by an earlier failure the loader reports first also gets through.
  Rationale: a broken lower layer, or an uninitialised base layer, must not block `--set` from writing or repairing a layer.
- **The hub-wide path keeps its strict `configengine.Load` check and adds the load check after it, over the board dir alone.**
  Rationale: the board-dir write is validated against the board file alone today, and the load check follows the same geometry.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads layer files; the load check calls the modules' loaders, which go through `configengine`.
- `PATTERN-told-geometry`: the load check takes the `configengine.Dirs` the caller already holds.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing `configcli` set tests; no new tier.
- Docs land in the same commit: `internal/configcli/doc.go`, `lyx config --help`, and `docs/overview.md` where it describes `--set`.

## Auto-mode assumptions

- A loader's error text is deterministic for the same files, so comparing before and after texts tells a new failure from an old one.
- No loader writes a file or spawns a process; some log a warning, which a second call repeats harmlessly.

## Open risks

- A loader that reads another module's file, such as loom reading `burler.yaml` and `models.yaml`, makes a write depend on that file too; the before/after rule keeps an already-broken neighbour from blocking it.
- A loader that changes to reject more values makes `--set` reject them too, which is the intent.

## Acceptance criteria

- `--set 'plan=["opus[medium]", "sonnet[high]"]'` on loom's local layer is refused, the error names `plan`, and the local file is byte-identical to before (or still absent).
- The same refusal holds on the worktree layer, and on a hub-wide or `--layer default` write, where nothing is committed.
- A valid per-round list on a per-round key, such as `plan_review`, is still written as a list.
- A `--set` onto a stack that already fails to load with the same error still writes.
- Every registered module has a load check set.

## Notes for the plan writer

- `internal/configcli/configcli.go`: `setModule`, `setLocal`; `internal/configcli/hubwide.go`: `setHubWide`, `snapshotFile`, `rollBack`, `validateHubFile`.
- `internal/configreg/configreg.go`: `Module`, `Modules()`.
