# Migrate key renames in the hub config and warn on unknown keys

After `loom-model-key-names` renamed the loom and darn keys, the hub's `_board/_lyx/config/*.yaml` kept the old names.
The new keys resolved from the template, so the hub's lens fans, fixer and reviewer models were silently lost.
`lyx config reconcile` never reached those files, and from `_board/` it treated them as a worktree layer, so `--apply` there would have stripped them.
Nothing warned that a hub file held keys the template no longer knows.

## What changes

- `lyx config reconcile --layer default` reconciles every module's file under the hub's board dir through `configsync.ReconcileHubWideAt`, the call the hub walk makes.
  Each module's `Migrate` hook renames old keys, a document holding both names keeps the new value and drops the old key, and keys the template lacks are removed.
  A dry run is the default and the report carries `layer: default` and the `seed` per module.
- `--apply --layer default` is guarded like `--set --layer default`: it is refused from a worktree that is not the prime and from a strand carrying a slug.
  Under the board write lock it dry-runs first, so a file that fails to parse or migrate leaves every file unwritten.
  It then writes, commits the files in `_board` and pushes.
  A write that fails part-way restores the board dir's config files.
  A commit failure and a push failure are reported apart, each naming the files left for the next board sync.
- `lyx config reconcile` run from the board dir is refused, with or without `--apply`, naming `lyx config reconcile --layer default` from the prime as the way forward.
  Any `--layer` other than `default` is refused.
- `configengine`'s load logs one `logger.Warn` for a default- or worktree-layer file holding units the template lacks, naming the module, file, keys and the reconcile that fixes it.
  Open-map entries and retired keys do not warn, nor does a `WholeEntries` spec.
  The warning is deduplicated per process on file and key list.
  The load still succeeds, so a binary change does not block runs before the reconcile.
- The `lyx config` help, the `missingBaseError` message, `internal/configcli/doc.go`, `internal/configengine/doc.go`, `docs/shared-libs/configengine.md` and `docs/overview.md` follow.
  The help page drops its per-round model-key example, which `doc.go` carries, to stay under the help length cap.

Closes #709

## Design decisions

- The hub walk (`internal/hubreconcile`) keeps running the migration after a binary change; the new verb is the operator's manual route to the same reconcile, with `SeedDefaults: true` so it also seeds an absent per-worktree repo default.
- The warning is a warning, not a refusal: the local layer already refuses unknown keys and is not covered.
- The guard bounds a mistaken write, not a determined agent; a dry run is never guarded.
- The hub's `preserved` old keys named in the issue are already gone: the hub walk for build `a273d69bc` removed them.
  This change writes no hub file itself; the new verb removes such keys from any hub that still holds them.

## To check by hand

Not run: a real hub with an old-key `loom.yaml` through `--layer default --apply` from the prime.
The commit-failure and push-failure branches of `reconcileDefaultLayer` have no test, since no commit seam exists to fake them.
The failed-apply integration test skips on Windows and as root.

Review findings to confirm are addressed:

- `internal/configcli/hubwide.go` `reconcileDefaultLayer`: a failure part-way through the apply must leave no file written, restored by `restoreBoardConfig`.
- `reconcileDefaultLayer`, final switch: a push failure alone must say the files are committed but not pushed.
- `internal/configcli/configcli.go` `runReconcile`: from the board dir with a non-default layer, the board-dir refusal must show first.
- `internal/configcli/configcli_integration_test.go`: the failed-apply path should be pinned.
- `internal/configcli/configcli.go` config help: preserved keys must point at `reconcile --layer default --apply`.
- `internal/configengine/config.go` `missingBaseError`: the comment and message must name `reconcile --layer default --apply` as a writer of the repo default.
- `internal/configengine/config.go` `warnUnknownKeys`: a `WholeEntries` spec must not warn; `Prune` still re-parses each layer on every load.
- `internal/configcli/configcli_integration_test.go`: a dry run from a task pair should exit 0 and leave the board `loom.yaml` unchanged.
