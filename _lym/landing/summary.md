# Keep unknown config keys in automatic hub reconciles

An automatic hub reconcile no longer removes a key that the running binary's template lacks.
Before, a binary older than the one that wrote a key stripped it from the committed board: an older build's reconcile removed `pull_request` from the beam config and `pr_reviewer` from landing.yaml, because its templates did not know them.

Now the binary-change walk, a clone and `lym fabric reconcile` remove only a key a retired key of its module covers.
They keep every other key the template lacks, in the board's base files and in each worktree file, and report it as `Preserved` in `configsync.Result`.
The walk also logs it as `preserved` when it writes the file.
`lym config reconcile --apply` is the one way to remove such a key, with `--layer default` for a board file.

Closes #2

## Design decisions

- `configsync.Options` and `configsync.HubWideOptions` gain `PruneUnknown`. Only `runReconcile` in `internal/configcli/configcli.go` and `reconcileDefaultLayer` in `internal/configcli/hubwide.go` set it.
- `pruneSparse` in `internal/configsync/configsync.go` first runs `yamlengine.Prune`; without `PruneUnknown` it then removes only the retired keys, via `configengine.DropRetired`, and returns the rest as preserved.
- A local override file is still pruned of every unknown key, since its load refuses one.
- A standalone worktree file, which is the base layer, is still fully reconciled.
- `configengine.dropRetired` is exported as `DropRetired`.
  It now also removes a mapping that its removal leaves empty, so a retired key does not leave an empty parent behind.
- The rule is recorded in `PATTERN-config-layers` and in the `internal/hubreconcile` package doc, and the `lym config reconcile` help text says this verb is the one way to remove such a key.

## To check

- `config-reconcile-prune-flag-untested` (design, MEDIUM): the tests do not clearly pin that `lym config reconcile` sets `PruneUnknown` in both the worktree path (`internal/configcli/configcli.go`) and the `--layer default` path (`internal/configcli/hubwide.go`). Check that a test would fail if either dropped it.
- The scenario from the issue was not run by hand across two machines with different lym builds.
