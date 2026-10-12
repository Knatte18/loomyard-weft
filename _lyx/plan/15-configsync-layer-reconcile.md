# Card 15 — configsync layer reconcile

**Create:**
- `internal/configsync#Options`
- `internal/configsync#HubWideOptions`
- `internal/configsync#TestReconcileAll_SparseLayers`

**Edit:**
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll` -> `func ReconcileAll(baseDir, boardDir string, opts Options) ([]Result, error)`
- `internal/configsync#ReconcileHubWideAt` -> `func ReconcileHubWideAt(boardDir, primeBaseDir string, opts HubWideOptions) ([]Result, error)`
- `internal/configsync#primeConfig`
- `internal/configsync#TestReconcileAll`
- `internal/configsync#TestReconcileAll_HubWideCopy`
- `internal/configsync#TestReconcileAll_Idempotent`
- `internal/configsync#TestReconcile_FailureNamesItsFile`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/configsync#TestReconcileHubWideAt_MigratesLegacyFabricConfig`
- `internal/configsync#reconcileFabricResult`
- `internal/hubreconcile#ensure`
- `internal/hubreconcile#reconcileHubWide`
- `internal/hubreconcile#reconcileWorktree`
- `internal/hubreconcile#snapshotConfig`
- `internal/hubreconcile/doc.go`
- `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`
- `internal/hubreconcile_test#runStaleBuildReconcile`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli_test#TestRunCLI_CleanHubScenario`
- `internal/fabriccli_test#TestRunCLI_CloneEndToEnd`
- `internal/fabriccli_test#TestRunCLI_AnchoredHubScenario`
- `internal/hubforge#NewHub`
- `internal/hubforge#cloneHub`
- `internal/hubforge#SeedConfig`
- `internal/hubforge/doc.go`
- `internal/hubforge#TestNewHub`
- `internal/orchcli#TestOrchIntegration_Refusals`
- `internal/loomcli/arm.go`
- `internal/loomcli/wiring.go`
- `internal/webstercli/wiring.go`
- `internal/configcli#runReconcile`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `PATTERN.md`
- `pattern/PATTERN-hub-wide-config.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/yamlengine#Prune`
- `internal/configengine#LayerFile`
- `internal/configengine#LocalFile`
- `internal/configengine#Layer`
- `internal/fabricengine#ConfigDirs`
- `internal/configreg#Module`
- `internal/hubreconcile#Ensure`

**Intent:**
Reconcile every layer by D2's rule: the base layer stays full, each layer above it is sparse.
`Options` holds `Apply bool`, replacing `ReconcileAll`'s bare flag; `HubWideOptions` holds `Apply bool`, replacing `ReconcileHubWideAt`'s bare flag, and `SeedDefaults bool`; `Result` gains a `Layer configengine.Layer` field naming the layer reported.

`ReconcileHubWideAt` now reconciles every module's file at the board dir, in registry order, each reported with `Layer` default: a hub-wide module's hub file as today, and, by D3, a per-worktree module's repo default as that module's base layer, full: a present default is migrated and reconciled (keys added and pruned) always; an absent one is seeded only under `SeedDefaults`, from the prime's file when that file exists and parses, as a `SeedsFromPrime` module is (D6), and otherwise from the template; a seed-only module's absent default is seeded once, under `SeedDefaults`, from the prime's file or the template, and a present one is never rewritten.
`hubreconcile`'s walk (`reconcileHubWide`) passes `SeedDefaults: true`; `fabriccli.runReconcile` and `CloneAndWire`'s own `ReconcileHubWideAt` call pass false and leave an absent default absent, which keeps D6's seed inside `hubreconcile.Ensure` (`PATTERN-binary-change-reconcile`).

A fresh hub still gets its defaults before any strict load needs them, through that same walk:
`CloneAndWire` ends, after wiring the junctions and committing the prime's config, by calling `hubreconcile.Ensure(hubreconcile.Geometry{BoardDir: res.BoardDir, WorktreePath: l.WorktreePath()}, hubreconcile.Options{})`.
The new hub has no build stamp, so the walk runs, seeds every absent per-worktree default from the prime or the template, commits them on the board and pushes it, and writes the stamp.
`CloneAndWire` never passes `SeedDefaults: true` itself and never seeds outside `Ensure`, the bound round 1 set; it appends a `KindFileWritten` entry for each per-worktree default absent before the call and present after, so the clone's mutation record covers the seeded files, and a walk failure returns as the clone's error with the clone left intact, the next start verb's walk seeding the defaults.
`Ensure` also counts a stamp naming the running build as stale when any per-worktree module's repo default is absent under the board dir, one stat per module beside the stamp read, so a default deleted after a walk is reseeded by the next start verb, which is the way forward card 2's strict `Load` names for it.
`internal/hubreconcile/doc.go`'s stamp section says so.

loom is told the hub dirs here, in the card that seeds its base layer: `internal/loomcli/wiring.go` and `arm.go` pass `fabricengine.ConfigDirs(location)` to `loomengine.LoadConfig` and `DiscussionSpec` in place of card 9's transitional `configengine.Dirs{Anchor: location.AnchorPath()}`, and webstercli passes it to `LoadConfig`.

A hubforge hub now carries the per-worktree repo defaults on its board and a build stamp from the clone's walk.
`cloneHub` removes that stamp after `CloneAndWire` returns, so a fixture hub keeps the stale stamp its tests start from and a start verb driven in a test walks it as on a binary change, while the seeded defaults stay committed.
`internal/hubforge/doc.go`'s seeding contract, `NewHub`'s doc and `SeedConfig`'s doc stop saying a clone arrives with every module's config materialised and committed in the weft prime: a clone arrives with each per-worktree module's repo default committed on the board, its prime carrying only the per-worktree config files the records repository already held, and `SeedConfig` writes a sparse worktree layer over those defaults.
`<BoardDir>/_lyx/config/` now holds both kinds of file, told apart by `configreg.Module.HubWide`; nothing else in the directory's handling assumes it holds hub-wide modules only, so `fabriccli.runReconcile`'s template-seed warning about uncarried types and labels is limited to hub-wide modules.

`ReconcileAll` reconciles a worktree's layers: standalone (`boardDir` empty) the worktree file is the base layer and keeps today's full reconcile; in a hub the worktree file is sparse: a present one has the module's `Migrate` hook run over it and its retired keys pruned with `yamlengine.Prune`, nothing is ever added, and an absent one stays absent.
The local file of every module, hub-wide ones included, is a sparse layer reconciled the same way and reported with `Layer` local; a seed-only module's worktree and local layers are never rewritten, pruned or migrated.
A hub-wide module's leftover worktree copy keeps today's `Retired` and `Divergent` result, with `Layer` worktree.
A file emptied by a prune is deleted, and its result reports it applied.

Callers follow: `hubreconcile.reconcileWorktree` commits only worktree-layer results and never a local file, and `snapshotConfig` also snapshots each module's local file so a failed worktree restores it; `CloneAndWire` records and commits only worktree-layer results; `configcli.runReconcile` adds a `layer` field to each module result.
`internal/configsync/configsync.go`'s file header says `ReconcileHubWideAt` covers every module's file at the board dir, hub files and per-worktree repo defaults, and `ReconcileAll` a worktree's sparse and local layers; the file stays out of `Edit:` beside its members, which `redundant-file-target` refuses.
The hub walk's existing integration scenarios are updated to the new shape: the board commit now carries the per-worktree defaults, and a pair's sparse files are pruned but never filled.
`TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` gains a case in which a stamp naming the running build sits beside loom's deleted repo default: `Ensure` walks, reseeds the default and commits it on the board.
The case deletes loom's default, never `batcher`'s, so `retiredKeyPresent` keeps reading a `batcher.yaml` whose default stays intact.
`TestRunCLI_CleanHubScenario` asserts a fresh clone's board carries every per-worktree module's repo default, committed.
Its `CloneConfigCommit_MutationRecordShape` step no longer expects one prime-side `KindFileWritten` per per-worktree module followed by a weft commit;
it expects one `KindFileWritten` entry per seeded repo default, each targeting that module's file under the board dir.
Its `CloneConfigCommit_WeftPrimeCleanAfterClone` step asserts the prime is clean and that `git ls-files` on the board lists every per-worktree module's repo default.
Its `CloneConfigCommit_OneCommitNotOnePerModule` step asserts a fresh clone's prime carries no `fabric clone: record module configs` commit, and that a second `ReconcileAll` applies nothing.

Its `CloneConfigCommit_PairInheritsConfigs` step no longer asserts an anchored `loom.yaml` on the pair's weft sibling; it asserts a pair forked from the clone resolves loom from the board's repo default: `loomengine.LoadConfig` over `fabricengine.ConfigDirs` of the pair loads with no worktree file present.

The other tests that read a per-worktree file a hubforge clone used to write follow the new clone output.
They were found by one search, which a later round reruns: every test under any build tag that reads `configengine.ConfigFile` or `ConfigFileRel` of a per-worktree module from a `hubforge` hub or a `CloneAndWire` result without seeding or committing it first and without tolerating its absence.
`TestRunCLI_CloneEndToEnd` stops asserting that `configengine.ConfigFile(weftBase, "loom")` exists and asserts the board carries loom's repo default instead, keeping its check that no per-worktree `board.yaml` sits under the weft base.
`TestRunCLI_AnchoredHubScenario`'s `CloneConfigCommit_AnchorScoped` step keeps its clean-prime check and asserts `git ls-files` on the board lists every per-worktree module's repo default, in place of `backend/<ConfigFileRel(m)>` in the prime.
`TestNewHub`'s `ConfigMaterializedWithoutSeeding` step asserts the board's loom repo default matches the registered template, not the prime's `loom.yaml`.
Its `SeedConfigRedundantSeedDoesNotFatal` step seeds loom once with `SeedConfig` before reading the file back.
The `hub config reconcile` subtest of `TestOrchIntegration_Refusals` writes the unparseable pair `loom.yaml` without reading an original first, and removes that file afterwards instead of restoring the old bytes.

`TestReconcileAll_SparseLayers` is table-driven over a temp board dir and anchor: a hub worktree file with a retired key is pruned and gains no template key, a below-floor loom value is raised by `Migrate`, an absent worktree file stays absent, a local file is pruned and reported with layer local, a seed-only module's present worktree and local files are left byte-identical, and an emptied file is deleted.
`TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains per-worktree default rows: seeded from the prime, from the template when the prime has none, a present default reconciled in place, and, with `SeedDefaults` off, an absent default left absent while a present one is still reconciled; the other configsync tests take `Options` or `HubWideOptions`.

`PATTERN.md`'s `PATTERN-hub-wide-config` entry and its background say that `<BoardDir>/_lyx/config/` also holds per-worktree modules' repo defaults, and that a hub-wide module also reads the reading worktree's local override, restricted to its `LocalKeys`.
The budget row for `internal/configsync` rises by one.

**ImpactSummary:** re-signs `ReconcileAll` and widens `ReconcileHubWideAt` to per-worktree defaults: the hub walk, fabric clone and reconcile, and `lyx config reconcile` follow; a clone ends with the walk that seeds the defaults, loom's callers move to the hub dirs, hubforge drops the clone's stamp, and the walk's integration scenarios are updated.
