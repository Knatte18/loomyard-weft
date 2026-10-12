# Card 7 — batcher layered load

**Create:**
- `plan:internal/batcher#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/batcher#Active` -> `func Active(dirs configengine.Dirs) (Batcher, error)`
- `internal/batcher#loadConfig` -> `func loadConfig(dirs configengine.Dirs) (config, error)`
- `internal/batcher#ProfileWeights` -> `func ProfileWeights(dirs configengine.Dirs, profileName string) (Weights, error)`
- `internal/batcher#retiredKeyError` -> `func retiredKeyError(file string, layer configengine.Layer, profileName, key string) error`
- `internal/batcher#profile`
- `internal/batcher#newCostFromProfile`
- `internal/batcher#profileWeights`
- `internal/batcher/doc.go`
- `internal/batcher_test#TestActive`
- `internal/batcher_test#TestMigrateConfig`
- `internal/batcher_test#TestProfileWeights`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#batchifierEntry`
- `internal/shedrecipe#loomPreflightEntry`
- `internal/shedrecipe#websterEntry`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#TestBatchifierEntry_RelativeAnchorPath`
- `internal/shedrecipe#TestSimpleEntries_UnderfilledEnv`
- `internal/shedrecipe#TestWebsterEntry_SeamFields`
- `internal/loomshed#batchifier`
- `internal/loomshed#NewBatchifier` -> `func NewBatchifier(name string, dirs configengine.Dirs) shedengine.ShedProducer`
- `internal/loomshed#batchifier.Call`
- `internal/loomshed#loomPreflightProducer`
- `internal/loomshed#NewLoomPreflight` -> `func NewLoomPreflight(name, statusPath, statusLockPath string, dirs configengine.Dirs) shedengine.ShedProducer`
- `internal/loomshed#loomPreflightProducer.checkBatcher`
- `internal/loomshed#retiredKeyWayForward`
- `internal/loomshed/doc.go`
- `internal/loomshed#websterProducer`
- `internal/loomshed#NewWebsterProducer` -> `func NewWebsterProducer(name string, dirs configengine.Dirs, run shedadapters.WebsterRunner, deps websterengine.RunDeps, commit func() error) shedengine.ShedProducer`
- `internal/loomshed#websterProducer.Call`
- `internal/loomshed#TestBatchifier_Call`
- `internal/loomshed#TestBatchifier_DistinctCausesYieldDistinctReasons`
- `internal/loomshed#TestCancellation_RealProducersReturnErrorNotStuck`
- `internal/loomshed#TestBatcherRows_StuckSurfacesTheBatcherError`
- `internal/loomshed#TestLoomPreflight_StuckSurfacesItsFailures`
- `internal/loomshed#TestGateRows_StopsNameAWayForward`
- `internal/loomshed#TestLoomPreflight_Call_BatcherConfig`
- `internal/loomshed#TestLoomPreflight_Call_CoherentSeedReportsDone`
- `internal/loomshed#TestLoomPreflight_Call_GotoReentryPassesAndLeavesStatusUnchanged`
- `internal/loomshed#TestLoomPreflight_Call_GotoThenLaterRowIsNotReentry`
- `internal/loomshed#TestLoomPreflight_Call_HalfFinishedWithoutGotoStuckNamesReentry`
- `internal/loomshed#TestLoomPreflight_Call_LockParentUncreatableReturnsError`
- `internal/loomshed#TestLoomPreflight_Call_ReentryWithOtherFailureStaysStuck`
- `internal/loomshed#TestLoomPreflight_Call_StuckReasonNamesTheFailures`
- `internal/loomshed#TestWebsterProducer_Call`
- `internal/loomshed#TestWebsterProducer_CommitsTheRunRecordOnDoneOnly`
- `internal/loomshed#TestWebsterProducer_PassesStuckReasonThrough`
- `internal/loomrecipe#wantProducerTable`
- `internal/testkit/envkit#FullEnv`
- `internal/configsync#TestReconcileAll`
- `internal/webstercli#seedPersistentPreRunConfig`
- `internal/loomcli/wiring.go`
- `internal/webstercli/wiring.go`
- `tools/tokencount/calibrate.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/configengine#Spec`
- `internal/configengine#Dirs`
- `internal/configengine#Layer`
- `internal/configengine#StackLayers`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Move batcher onto the hub stack: `Active`, `loadConfig` and `ProfileWeights` take `configengine.Dirs` and load through `configengine.LoadOrTemplate(dirs, ConfigSpec())`.
`ConfigSpec` returns batcher's `Spec` with `ConfigOpenMaps()` and `Categories` exactly as the overview's "Per-key classification" table lists its keys, `active` and the open map `profiles` both none.
Because the merged document carries every layer's keys, a retired `batcher.yaml` key in any layer reaches the retired-key refusal, which is how Loom-Preflight's retired-key check sees every layer (the record's note on Loom-Preflight); `profiles` is an open map and is taken whole from the highest layer holding it.
The retired-key error names the layer file that carries the key, `configengine.StackLayers(dirs, ConfigSpec())`'s `Origins["profiles"]`, so batcher never picks the winning layer itself, and its way forward follows that layer: `lyx config reconcile --apply` for a worktree or local file, and for the repo default, which only the hub-wide reconcile rewrites, `run "lyx fabric reconcile"` followed by the clause `; a session lyx refuses the verb from reports status: FAILED and the orch runs it`.
`loadConfig` records that file and layer on each decoded `profile` in two unexported fields with no YAML tag, and the two retired-key sites, `newCostFromProfile` and `profileWeights`, keep their signatures and pass the profile's file and layer to `retiredKeyError`.
`loomshed.retiredKeyWayForward` keeps completing either way forward with `, then re-step`, its doc naming both; batcher's and loomshed's `doc.go` and `contracts/specs/refusal-spec.md`'s rows "config file content invalid" and "Loom-Preflight batcher" name the default-layer way forward beside `lyx config reconcile --apply`.

The three loomshed rows that gate on batcher are told the dirs instead of a bare anchor: `NewBatchifier`, `NewLoomPreflight` and `NewWebsterProducer` take a `configengine.Dirs` in place of their anchor or config-base string, the `batchifier`, `loomPreflightProducer` and `websterProducer` structs hold it in place of their string field, and their logs name the dirs' anchor.
`shedrecipe.Env` gains a `ConfigDirs configengine.Dirs` field, documented as read by Batchifier, LoomPreflight and Webster; `batchifierEntry`, `loomPreflightEntry` and `websterEntry` validate `env.ConfigDirs.Anchor` as an absolute root, naming the field `ConfigDirs.Anchor` in their refusal, and pass `env.ConfigDirs`.
`internal/loomcli/wiring.go` sets `c.env.ConfigDirs = fabricengine.ConfigDirs(location)` beside `c.env.AnchorPath`, and calls `batcher.Active` with the same dirs; webstercli passes `fabricengine.ConfigDirs(location)` in a hub and `configengine.Dirs{Anchor: stateDir}` standalone; `envkit.FullEnv` fills `ConfigDirs` with its anchor dir; `tools/tokencount`'s `Calibrate` passes `configengine.Dirs{Anchor: configDir}`, so the calibration tool keeps reading the one dir it is given.

`TestActive` gains a hub-stack row over a temp board dir and anchor, an `active` set only in the repo default selecting its profile beneath a worktree file lacking the key, a row in which a worktree file's `profiles` replaces the default's whole, and a row in which a retired key in the repo default fails naming the default file and `lyx fabric reconcile`; the shedrecipe and loomshed tests listed take the new field and arguments, and `TestBatchifierEntry_RelativeAnchorPath` asserts the relative `ConfigDirs.Anchor` refusal.

**ImpactSummary:** re-signs `batcher.Active`, `ProfileWeights` and three loomshed constructors and adds one `shedrecipe.Env` field: loomcli, webstercli, tokencount, envkit and the loomshed and shedrecipe tests follow.
