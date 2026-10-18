# Decision record — config-migrate-hub-layer

## Goal

A key-rename migration reaches the files under `<BoardDir>/_lyx/config/` (hub files and per-worktree repo defaults) through a hub-scoped `lyx config reconcile`, which removes the old keys.
A default- or worktree-layer config file holding keys its template no longer knows is reported loudly when it is loaded.

## Scope

- `lyx config reconcile --layer default` reconciles every module's file at the board dir through `configsync.ReconcileHubWideAt`: a dry run by default, `--apply` writes and commits in `_board`.
- `lyx config reconcile` run with the board dir as its base is refused with the way forward, so it never treats hub files as a worktree layer.
- `configengine`'s load warns once per process for a default- or worktree-layer file holding units the template lacks.
- Docs: `internal/configcli/doc.go`, `internal/configengine/doc.go`, the reconcile command's help, `docs/shared-libs/configengine.md`.
- Out of scope: the hub walk (`internal/hubreconcile`), which already runs the same migration over the board dir at the first start verb after a binary change, and the deploy script.

## Decisions

### D1: `lyx config reconcile --layer default` reconciles the board dir

`--layer` on `reconcile` takes only `default`; any other value is refused naming `default` as the one value.
It runs `configsync.ReconcileHubWideAt(boardDir, primeAnchor, {Apply, SeedDefaults: true})`, the call the hub walk makes, so each module's `Migrate` hook renames old keys, a document already holding the new key keeps it and drops the old one, and the prune removes keys the template lacks.
The prime anchor is the main code worktree's anchor from `fabricengine.CodeWorktrees`.
The envelope reports one entry per module with `layer: default` and the `seed` it started from, in the shape the worktree reconcile reports.
Rationale: the board dir is the base layer of every module's stack, and the walk is the only code that reconciles it today, gated on a build stamp the operator cannot trigger by hand.

### D2: `--apply` on the default layer is guarded and committed

`--apply --layer default` runs `hubWriteRefusal`, the guard of a `--set --layer default` write: refused from a worktree that is not the prime and from a session whose strand name carries a slug, with the board left unchanged.
Under the board write lock (`fabricengine.Bolt.CommitWritten`) it runs the reconcile as a dry run first, so a parse failure in any module writes nothing, then applies it and commits each applied module file and pruned legacy file in `_board`, then pushes.
Bound: the guard stops a mistaken write as the `--set` guard does, not a determined agent; a dry run is never guarded, since it writes nothing.

### D3: reconcile from the board dir is refused

When the resolved base dir is the hub's board dir, `lyx config reconcile` with or without `--apply` and `--layer` is refused before reading any file, naming the way forward: `lyx config reconcile --layer default` from the hub's prime worktree.
Rationale: from `_board/` the worktree reconcile would strip hub files as a sparse layer and retire the hub-wide modules' files as leftover copies.

### D4: unknown keys warn once per process at load

`configengine`'s load prunes each stacked default- and worktree-layer file against the template with `yamlengine.Prune` and the spec's open maps, drops the spec's `RetiredKeys`, and logs one `logger.Warn` for a file with units left over, naming the module, the file, the keys and the way forward:
`lyx config reconcile --layer default --apply` from the hub's prime for a default-layer file, `lyx config reconcile --apply` in the worktree for a worktree file.
A file and key set already warned in the process is not warned again, so a loader in a poll loop logs it once.
The load still succeeds: a refusal would stop every run between a binary change and the reconcile that fixes the file.
The local layer already refuses unknown keys and is not covered.

### D5: the hub's preserved old keys are already gone

The hub walk for build `a273d69bc` (board commit `aa9f100c5`) removed every `preserved` old key from the hub's `loom.yaml` and `darn.yaml`, and the files hold none today.
The operator decision is met by that run of the migration; this task writes no hub file itself, and D1's migration removes such keys from any hub that still holds them.

## Constraints

- `PATTERN-config-layers`: only `configengine` stacks layers; the warning lives in its load.
- `PATTERN-hub-wide-config`: the default-layer reconcile reads and writes only `<BoardDir>/_lyx/config/`.
- `PATTERN-cliwire-sole-wiring` and `PATTERN-cwd-resolution`: `configcli` resolves geometry through `lyxcwd` as it does today.
- `PATTERN-log-levels`: the warning is deduplicated so no loop floods stderr or the trace.
- `PATTERN-test-economy` and `PATTERN-test-speed`: new cases extend the existing reconcile and load tests.
- `PATTERN-binary-change-reconcile`: the walk after a binary change stays `hubreconcile.Ensure`'s; the new verb is an operator-run reconcile.

## Auto-mode assumptions

- `SeedDefaults: true` matches the walk, so a manual reconcile also seeds an absent per-worktree repo default.
- The warning goes to `logger.Warn`, which reaches stderr and the durable sink at the default verbosity; that is "reported loudly".

## Open risks

- A long-lived process that loads config before and after a hub file changes warns again only for a new key set.
- A failed write after the dry run leaves written files uncommitted; the report says so and the next board commit carries them, as a failed `--set` commit does.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - apply-failure-leaves-files-unrestored (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- `lyx config reconcile --layer default` from the prime, over a board `loom.yaml` holding an old key name, lists the rename under `migrated` with `layer: default` and writes nothing.
- `--apply` rewrites the board file to the new name and commits it in `_board`.
- A board file holding both an old and a new key name keeps the new value and drops the old key on `--apply`.
- `--apply --layer default` from a pair, or from a slug-bearing strand, is refused and leaves the board file unchanged.
- `lyx config reconcile` from the board dir is refused naming the way forward, and writes nothing.
- `--layer worktree` or `--layer local` on reconcile is refused.
- Loading a module whose board file holds a key the template lacks logs one warning naming the file and the key, and a second load in the same process logs none; an open-map entry and a retired key log none.
