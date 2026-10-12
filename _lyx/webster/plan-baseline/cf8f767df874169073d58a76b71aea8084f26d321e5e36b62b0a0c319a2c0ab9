# Card 12 — fabric layered load

**Create:**
- `plan:internal/fabricengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/fabricengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `internal/fabricengine#WiredNames` -> `func WiredNames(dirs configengine.Dirs) ([]string, error)`
- `internal/fabricengine#PathspecNames` -> `func PathspecNames(dirs configengine.Dirs) ([]string, error)`
- `internal/fabricengine#junctionNames` -> `func junctionNames(dirs configengine.Dirs) ([]string, error)`
- `internal/fabricengine#RepoWiredNames`
- `internal/fabricengine#Fabric.Commit`
- `internal/fabricengine#commitPendingRecords`
- `internal/fabricengine#Topology.RemoveRefusal`
- `internal/fabricengine_test#TestLoadConfig`
- `internal/fabricengine_test#TestHubReserved_BoardExcludedFromPathspecRoutes`
- `internal/fabricengine#TestJunctionNames_NoFallbackOnLoadFailure`
- `internal/fabricengine_test#firstWiredJunction`
- `internal/fabricengine_test#driveOwnershipGateRefusal`
- `internal/fabricengine_test#firstWiredJunctionLink`
- `internal/fabricengine_test#unwireJunctionsCase`
- `internal/fabricengine_test#newClonedHubFixture`
- `internal/fabricengine_test#resolvedDefaultRoutingNames`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli#runAdd`
- `internal/fabriccli#runCheckout`
- `internal/fabriccli#runCleanupWithFlags`
- `internal/fabriccli#runList`
- `internal/fabriccli#runPairs`
- `internal/fabriccli#runPruneWithFlags`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli#runRemoveWithFlag`
- `internal/fabriccli#addWeftVerbs`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestBattenIntegration_CreateRow_IncompletePairRemedyWorksVerbatimOnAPrefixedHub`
- `internal/pairteardown#Teardown.topology`
- `internal/hubforge#assertRealHub`
- `internal/hubforge#TestNewHub_TeardownSurvivesCorruptHub`

**Delete:**
- `internal/fabricengine#repoWideFabricBase`

**Uses:**
- `internal/configengine#Spec`
- `internal/configengine#Dirs`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Move fabric, the last hub-wide module, onto the hub-wide stack: `LoadConfig`, `WiredNames`, `PathspecNames` and `junctionNames` take `configengine.Dirs` and load strictly through `configengine.Load(dirs, ConfigSpec())`.
`ConfigSpec` is hub-wide with `Categories` as the overview's table lists: `branch_prefix` and `pathspec` cross-pair, since each shapes a pair at creation and is the one repo-wide fact every pair converges to, so a pair's `fabric.local.yaml` holding any key fails the load naming the file and the key.

Inside fabricengine, `RepoWiredNames(l)` keeps its signature and `Fabric.Commit` keeps its own; both pass `ConfigDirs(l)`, which makes `repoWideFabricBase` dead, so this card deletes it.
`commitPendingRecords` and `Topology.RemoveRefusal` pass `ConfigDirs(l)` in place of `BoardDir(l.HubPath)`.
The fabriccli verbs pass `fabricengine.ConfigDirs(l)`; `CloneAndWire`, which holds the clone's result rather than a location, passes `configengine.Dirs{Anchor: res.WeftBase, Board: res.BoardDir}`; battencli's `wire` and pairteardown's `topology` pass the dirs of the location they hold.
Tests pass `configengine.Dirs{Board: boardDir}` where they read a hub's fabric file, keeping today's reads.

`TestLoadConfig` gains a row over a temp board dir and anchor in which a local `pathspec` fails with `ErrInvalid` naming the local file and the key; the other listed tests take the new argument.

**ImpactSummary:** re-signs four fabricengine loaders and deletes their dead base helper: fabriccli, battencli, pairteardown and hubforge follow, all reading the same hub file as before, even when the reading worktree's `_lyx/` is absent, as card 2's `TestLoad_LayerStack` row pins.
