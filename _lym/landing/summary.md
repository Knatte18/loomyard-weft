# Reconcile migrates profiles.yaml for renamed and missing recipes

A hub whose `profiles.yaml` was seeded before the `loom` to `tapestry` recipe rename stopped every tapestry run at Seed-Child.
`lym config reconcile` left the file alone, because `profiles` is a seed-only module that reconcile never rewrites, so only hand-editing each profile with `lym config profiles --set` fixed it.

Reconcile now runs two hooks over a present `profiles.yaml` and still never fills or prunes it.

- `runprofile.MigrateConfig` renames a recipe block, in each profile and under `defaults`, held under a recipe's former name to the current name, keeping the operator's value and the file's comments.
  A table in `internal/runprofile/migrate.go` (`recipeRenames`) lists the renames; it holds `loom` to `tapestry` today.
- `runprofile.LackingTemplateBlocks` lists each recipe block (such as `sampler`) that the template holds for a profile and the hub's profile of that name lacks.
  Reconcile reports it under a new `lacking` field of the envelope and logs one Warn per module in the hub reconcile walk, with `lym config profiles` as the way forward.
  Nothing is added, since the file is the operator's.

Seed-Child now refuses a profile, or a `defaults` entry, that holds a recipe only under its former name, and names `lym config reconcile --layer default --apply` as the way forward.
`Profiles.Default` now returns an error for that case, and `prepareRunSettings` in `battencli` wraps it in `ErrInvalidRunSettings`.
Before, a task whose default sat under the former name quietly ran on the hub's own config.

Closes #29

## Design decisions

- `configreg.Module` gains a `Lacking` hook, and the `SeedOnly` contract changes from "never rewritten" to "never filled or pruned".
  `configsync.ReconcileHubWideAt` runs `Migrate` and `Lacking` over a present seed-only file, or over the prime's copy that seeds an absent default, and writes only when `Migrate` rewrote something.
  An absent seed-only file with no prime copy is still seeded whole from the template.
  This also drops the `Seed: hub` marker that the reconcile envelope gave a present seed-only per-worktree module, since such a file now goes through the same path.
- When a profile holds both a former block and the current one, the former block is deleted only when it equals the current block.
  A differing pair is left untouched and reported as `<entry>.<from>: differs from <entry>.<to>` under `lacking`, so the operator settles it by hand.
  This keeps the migration from ever dropping an operator value silently.
- The migration edits the yaml node tree, so comments and unrelated keys survive.
  A document that does not parse, or whose root is not a mapping, fails with `configengine.ErrInvalid`.
- The `SeedOnly` doc comments, `profiles` template header and the docs of `runprofile`, `configcli`, `configengine` and `hubreconcile` are updated to the new contract.

## To check by hand

- Run `lym config reconcile --layer default` (dry run, then `--apply`) from the prime of a hub whose `profiles.yaml` still holds `loom` blocks and `defaults.loom`.
  The blocks should be renamed with their values and comments kept, and `lacking` should list the missing `sampler` blocks.
- On such a hub before reconcile, start a tapestry task and confirm Seed-Child names the reconcile as its way forward.
  This path is covered by unit tests only.
