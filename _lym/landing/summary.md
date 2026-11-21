# Remove compatibility and migration code no hub needs any more

The code carried one-off migrations, key renames and readers for formats that no hub still holds.
The operator confirmed that no hub needs any of it, so this change deletes it module by module.
It removes about 19,000 lines and adds about 2,100, mostly in tests and docs.

## What changes

- **Deleted migrations and compatibility readers.**
  Examples are the `landingshed` retired `Merge-In` row name, the scratch-to-records attempt migration and the key-rename migration.
  Others are `loomengine.MigrateConfig` with its producer-key, skip-merge-review, fan-review and review-key renames,
  and `configsync`'s `movekeys.go` and its `Migrate` option.
  `dobbyengine/upgrade.go`, `fabricengine/portals.go`, `retiredlaunchers.go`, `pawlengine/retiredmove.go`,
  `runprofile/migrate.go`, `boardengine/legacy.go` and `migrate.go`,
  the `yamlengine` rename, drop and floor helpers, and the `reinterpret.go` files of `agentconfig`, `creelconfig` and `shuttleengine` go too.
  `driftengine` loses its stored old-Drift scope readers (partial scope, compare, waiver-render and acceptance fields),
  `diffmode.go`, `dirbase.go` and `accepted.go`.
- **`lym board retire-legacy` is gone.**
  It was a one-off migration of `tasks.json` and `notes.json` into `board.json`, and every hub has run it.
- **Refusals that point to the new name stay.**
  A retired key or verb still fails with a way forward.
  `loomengine` now refuses a `plan` key (and the other tapestry plan keys) in run settings and names the `tapestry` module as the place for it.
  `batcher` offers `lym config reconcile --layer default --apply` as the way forward only for a profile the template names.
- **Reconcile no longer strips or moves.**
  `configsync` drops `Migrate` and `StripBase`, the sparse-marker bookkeeping and the settle-pair warnings.
  `hubreconcile` walks with the plain `Apply` and `SeedDefaults` options.
  `applyReinterpretations` takes its module list as an argument so a test can pin it, with `reinterpretationModules` as the production seam.
- **Fabric.**
  A stale-anchor marker is committed in the prime's weft worktree, and the remedy for it is adoptable.
  The recorded-merge guard stops evaluating the weft side.
- **Docs, patterns, recipes and the sandbox suite** drop their descriptions of the deleted migrations.
  `migrateRunSettings` is renamed `validateRunSettings`.
- **Tests.**
  Tests of the deleted paths go.
  New tests pin the surviving refusals, the registered shuttle engine modules,
  the walk-after-failed-reinterpretation case and the sibling walk over real hubs.

## Design decisions

- A hit counted as a refusal when it rejects an old name or key and points to the new one, which is cheap, so it stayed.
  It counted as migration when it rewrote or read data in the old shape, so it went.
- A hub that still holds an old key or file shape is now refused, not rewritten.
  The operator's confirmation that no such hub exists is the basis for this.

## To check by hand

- `driftengine`: the compatibility readers for the old Drift record shape were removed in stages.
  Confirm that no reader of the old scope, compare, waiver-render or acceptance fields remains under `internal/driftengine`,
  and that no remaining compatibility reader sits in another module.
- No regression check ran for this task, and nothing was run beyond the repository's own gates.
  Run `lym board` and `lym config reconcile` once against a real hub to confirm that neither reaches a removed path.

## Merge danger

**Door:** two-way
**Blast radius:** config
