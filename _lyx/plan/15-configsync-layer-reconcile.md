# Card 15 — configsync layer reconcile

**Create:**
- `plan:internal/configsync#Options` -> `type Options struct`
- `plan:internal/configsync#TestReconcileAll_SparseLayers` -> `func TestReconcileAll_SparseLayers(t *testing.T)`

**Edit:**
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll` -> `func ReconcileAll(baseDir, boardDir string, opts Options) ([]Result, error)`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#primeConfig`
- `internal/configsync#TestReconcileAll`
- `internal/configsync#TestReconcileAll_HubWideCopy`
- `internal/configsync#TestReconcileAll_Idempotent`
- `internal/configsync#TestReconcile_FailureNamesItsFile`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/configsync#reconcileFabricResult`
- `internal/hubreconcile#reconcileWorktree`
- `internal/hubreconcile#snapshotConfig`
- `internal/hubreconcile/ensure_integration_test.go`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli_test#TestRunCLI_CleanHubScenario`
- `internal/configcli#runReconcile`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `PATTERN.md`
- `pattern/PATTERN-hub-wide-config.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/yamlengine#Prune`
- `plan:internal/configengine#LayerFile`
- `plan:internal/configengine#LocalFile`
- `plan:internal/configengine#Layer`
- `internal/configreg#Module`

**Intent:**
Reconcile every layer by D2's rule: the base layer stays full, each layer above it is sparse.
`Options` holds `Apply bool`, replacing `ReconcileAll`'s bare flag; `Result` gains a `Layer configengine.Layer` field naming the layer reported.

`ReconcileHubWideAt` now reconciles every module's file at the board dir, in registry order, each reported with `Layer` default: a hub-wide module's hub file as today, and, by D3, a per-worktree module's repo default as that module's base layer, full: a present default is migrated and reconciled (keys added and pruned); an absent one is seeded from the prime's file when that file exists and parses, as a `SeedsFromPrime` module is (D6), and otherwise from the template; a seed-only module's absent default is seeded once from the prime's file or the template, and a present one is never rewritten.
`<BoardDir>/_lyx/config/` now holds both kinds of file, told apart by `configreg.Module.HubWide`; nothing else in the directory's handling assumes it holds hub-wide modules only, so `fabriccli.runReconcile`'s template-seed warning about uncarried types and labels is limited to hub-wide modules.

`ReconcileAll` reconciles a worktree's layers: standalone (`boardDir` empty) the worktree file is the base layer and keeps today's full reconcile; in a hub the worktree file is sparse: a present one has the module's `Migrate` hook run over it and its retired keys pruned with `yamlengine.Prune`, nothing is ever added, and an absent one stays absent.
The local file of every module, hub-wide ones included, is a sparse layer reconciled the same way and reported with `Layer` local; a seed-only module's worktree and local layers are never rewritten, pruned or migrated.
A hub-wide module's leftover worktree copy keeps today's `Retired` and `Divergent` result, with `Layer` worktree.
A file emptied by a prune is deleted, and its result reports it applied.

Callers follow: `hubreconcile.reconcileWorktree` commits only worktree-layer results and never a local file, and `snapshotConfig` also snapshots each module's local file so a failed worktree restores it; `CloneAndWire` records and commits only worktree-layer results; `configcli.runReconcile` adds a `layer` field to each module result.
The hub walk's existing integration scenarios are updated to the new shape: the board commit now carries the per-worktree defaults, and a pair's sparse files are pruned but never filled.

`TestReconcileAll_SparseLayers` is table-driven over a temp board dir and anchor: a hub worktree file with a retired key is pruned and gains no template key, a below-floor loom value is raised by `Migrate`, an absent worktree file stays absent, a local file is pruned and reported with layer local, a seed-only module's present worktree and local files are left byte-identical, and an emptied file is deleted.
`TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains per-worktree default rows: seeded from the prime, from the template when the prime has none, and a present default reconciled in place; the other configsync tests take `Options`.

`PATTERN.md`'s `PATTERN-hub-wide-config` entry and its background say that `<BoardDir>/_lyx/config/` also holds per-worktree modules' repo defaults, and that a hub-wide module also reads the reading worktree's local override, restricted to its `LocalKeys`.
The budget row for `internal/configsync` rises by one.

**ImpactSummary:** re-signs `ReconcileAll` and widens `ReconcileHubWideAt` to per-worktree defaults: the hub walk, fabric clone and reconcile, and `lyx config reconcile` follow, and the walk's integration scenarios are updated.
