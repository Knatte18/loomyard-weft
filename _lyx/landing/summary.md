# Reconcile the board dir's config by hand; warn on unknown keys at load

A key-rename migration such as `loom-model-key-names` reached the files under `<BoardDir>/_lyx/config/` only through the hub walk, which runs at the first start verb after a binary change.
`lyx config reconcile` in the prime covered the worktree's own and local files alone.
Run from `_board/`, it treated the hub's files as a sparse worktree layer, so `--apply` would have stripped them and retired the hub-wide modules' files.
Until something reconciled the hub file, a renamed key lost its value with nothing said: no loader reads a key the template lacks, and the load filled the new name from the template.

## What changes

- `lyx config reconcile --layer default` reconciles every file under the board dir: the hub-wide modules' files and each per-worktree module's repo default.
  It makes the same `configsync.ReconcileHubWideAt` call as the hub walk (`SeedDefaults: true`), seeded from the prime's anchor.
  Each module's `Migrate` hook renames old keys.
  A document already holding the new name keeps it and drops the old key.
  Keys the template lacks are pruned.
  The envelope matches the worktree reconcile's, with `layer: default` and a new `seed` field.
- `--apply` with it uses the guard a `--set --layer default` write uses: it is refused from a non-prime worktree and from a slug-bearing strand.
  It holds the board write lock (`Bolt.CommitWritten`) and dry-runs first, so a file that fails to reconcile leaves every file unwritten.
  Then it writes, commits the written files in `_board` under `config: reconcile the board dir's files`, and pushes.
  A dry run is never guarded.
- `lyx config reconcile` is refused when the resolved worktree is the board dir.
  `--layer` values other than `default` are refused too.
  Both refusals name the way forward.
- `configengine`'s `Load` and `LoadOrTemplate` still load a default- or worktree-layer file that holds units the template lacks.
  They now log a Warn naming the module, the layer, the file, the keys and the reconcile that removes them.
  Open-map entries and the spec's `RetiredKeys` are left out.
  The warning goes out once per file and key list in a process (`sync.Map`), so a loader in a poll loop does not flood stderr or the trace.
  The local layer already refuses such units.
  Seed-only modules load through `StackLayers` and get no warning.

## Design decisions

- `--layer default` on `reconcile` reuses the flag `lyx config <module>` already takes for the board dir.
  Making the bare reconcile also cover the board would have a pair's dry run report changes it can never apply.
- No landing or deploy step was added.
  The hub walk at the next start verb already runs this migration.
  For the window before that walk, the load warning names the manual route.
- The warning does not refuse the load.
  A refusal would stop every run between a binary change and the reconcile.
- The operator decision asked to remove the `preserved` old keys that `loom-model-key-names` left in the hub's `loom.yaml` and `darn.yaml`.
  The hub walk for build `a273d69bc` (board commit `aa9f100c5`) already removed them, and neither file holds any today, so this change writes no hub file.
  The new verb removes such keys from any hub that still holds them.
  The integration test pins this: a board `loom.yaml` holding `merge_fan` beside `merge_reviewer_fan` loses `merge_fan`.

## Tests

- `TestLoad_LayerStack` gains a row with unknown keys in the default and worktree layers.
  Over both loaders it expects exactly one warning per file, naming the keys and the way forward.
  The existing retired-key and open-map rows now also assert that no warning is logged.
- `TestConfigOverRealHub` gains a scenario on a real hub.
  The dry run of `--layer default` reports the `webster_fan` and `merge_fan` rewrites and writes nothing.
  `--apply` from the pair, reconcile from `_board`, and `--layer local` are each refused, and the file stays unchanged.
  `--apply` from the prime writes `webster_reviewer_fan`, drops both old keys and commits only `loom.yaml` in `_board`.

## To check by hand

On the live hub, `lyx config reconcile --layer default` from the prime should report no `migrated` or `removed` entries, since the walk already reconciled the board.
