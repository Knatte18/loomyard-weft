# Card 3 — Reconcile every hub-wide module at the board dir, seeding board.yaml from the prime

**Create:**
- `internal/configsync#ReconcileHubWideAt`

**Edit:**
- `internal/configsync/configsync.go`
- `internal/configsync/configsync_test.go`
- `internal/fabriccli/clone.go`
- `internal/fabriccli/fabric.go`
- `internal/fabriccli/cli_test.go`
- `internal/hubforge/seed.go`
- `internal/hubforge/doc.go`

**Uses:**
- `internal/fabricengine#Bolt.CommitWritten`
- `internal/fabricengine#PrimeName`
- `internal/configreg#Module`
- `internal/configengine#ConfigFileRel`

**Intent:**
Replace the fabric-only `ReconcileFabricAt` with `ReconcileHubWideAt`, which reconciles every module whose `configreg.Module.HubWide` is set, in registry order, at `configengine.ConfigFile(boardDir, name)` (D4), returning one `Result` per module.
`primeBaseDir` is the prime worktree's anchor dir, told by the CLI caller (`PATTERN-told-geometry`); an empty value means no prime is resolvable.

Per module:
- A present hub file is the reconcile input; it is never replaced, and reconcile only adds and prunes template keys, as today.
- An absent hub file of a module that declares `OpenMaps` (`board`) is seeded from `configengine.ConfigFile(primeBaseDir, name)` when that file exists, is non-empty and parses.
  A module with no open maps (`fabric`) is never seeded from the prime: a per-worktree `fabric.yaml` was edited by `lyx config fabric` but never read, so its values were never in effect (D5).
- Otherwise `fabric` keeps its legacy `warp.yaml`/`weft.yaml` fold-in from the board dir, unchanged, and every other module seeds from its template.
- Reconcile goes through `yamlengine.Reconcile` with the module's `OpenMaps`, so `board`'s `types` and `labels` are carried whole.
- With `apply`, the file is written when it was absent or changed, creating the config dir, and the migrated legacy files are removed exactly as `ReconcileFabricAt` does today.

`Result` gains a `Seed` field naming the reconcile input: `"hub"`, `"prime"`, `"legacy"` (the `fabric` fold-in, reported together with `MigratedFrom`) or `"template"`, set on a dry run too.
The seed reads one file, only when the hub file is absent, and only from the prime, so a pair's possibly stale copy is never read.

`lyx fabric reconcile` (`runReconcile` in `fabric.go`) calls `ReconcileHubWideAt` where it calls `ReconcileFabricAt` today.
It resolves the prime as `filepath.Join(l.HubPath, <fabricengine.PrimeName(l)>, l.AnchorRel)`; a `PrimeName` failure means no prime, not an error.
The reconcile runs inside `fabricengine.NewBolt(BoardDir).CommitWritten`, whose `write` returns `configengine.ConfigFileRel` of every applied module and of every removed legacy file, so the files are written and committed in `_board` under the board write lock; then `Bolt.Push` runs.
A commit or push failure does not change the exit code: it is reported under a new `hub_config_detail` envelope key and leaves the file for the next board sync, mirroring the warp-binding backfill.
The envelope gains `hub_config`, one `{module, seed, applied}` object per result.
For a module with open maps seeded from `"template"` it also gains a `warnings` entry; `fabric` seeded from its template raises none, since it has no vocabulary to carry.
The warning names the hub file and says the prime held no copy, so custom types and labels were not carried and must be added by editing that file.
Each applied file appends `KindFileWritten` and a landed commit appends `KindCommitCreated`, in execution order, as the handler records today.

`CloneAndWire` (`clone.go`) calls `ReconcileHubWideAt(res.BoardDir, res.WeftBase, true)` silently, with no warning, since no vocabulary can predate a fresh clone.
It records one `KindFileWritten` per applied result, and keeps its existing whole-repo `Bolt.Commit`, which already commits every file the fresh clone wrote.

Update `ReconcileAll`'s doc, the comments in `runReconcile` (`fabric.go`) and the `hubforge` `seed.go`/`doc.go` comments, which name `ReconcileFabricAt`, and the `configsync.go` header so it lists the hub-wide reconcile beside the per-worktree one.

Coverage:
- `configsync_test.go`: retarget `TestReconcileFabricAt_MigratesLegacyFabricConfig` to `ReconcileHubWideAt`, reading the `fabric` result.
  Add one table-driven test over the `board` result:
  - hub absent with a prime copy carrying a custom type and label → hub file equals the prime's open maps, `Seed == "prime"`;
  - hub absent and no prime copy → template, `Seed == "template"`;
  - hub present and a differing prime copy → hub file unchanged, `Seed == "hub"`;
  - for the `fabric` result: hub absent with a prime `fabric.yaml` → template, prime copy ignored; hub absent with legacy `warp.yaml` → `Seed == "legacy"`;
  - a dry run writes nothing in any row.
- `fabriccli/cli_test.go` (integration): extend `TestRunCLI_Reconcile_HealsMissingRepoWideConfig`.
  Delete the hub `board.yaml` too, and seed the prime's per-worktree `board.yaml` with a custom type through `hubforge.SeedConfig`.
  Assert that reconcile writes the hub file with that type, that `hub_config` names `board` with seed `prime`, and that the hub file is committed (clean in `git status` of `h.BoardDir()`).
  Add a row where the prime copy is removed as well, asserting seed `template` and a `warnings` entry naming the hub file.

**ImpactSummary:** `ReconcileFabricAt` is removed and its two fabriccli callers move to `ReconcileHubWideAt`; `fabric reconcile` gains a `_board` commit and new envelope keys, and clone also materializes the hub `board.yaml`.

**Commit:** 3: configsync: reconcile hub-wide modules at the board dir, seeding board.yaml from the prime
