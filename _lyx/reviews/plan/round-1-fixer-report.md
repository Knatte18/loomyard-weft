# Fixer report — plan, round 1

Every finding's premise held against the code and the record; each is fixed as proposed.

## Changes

- **F1** (card 2, card 12): the `_lyx/` probe runs on the base layer's directory, `Board` when set, else `Anchor`; an absent `_lyx/` under `Anchor` while `Board` is set means the worktree and local layers are absent.
  `LoadOrTemplate` degrades only on an absent base-layer `_lyx/` or base file.
  `TestLoad_LayerStack` gains the two `Dirs{Anchor: <dir without _lyx/>, Board: <initialised board dir>}` rows (hub-wide strict `Load`; per-worktree repo-default key under `Load` and `LoadOrTemplate`).
  Card 12's ImpactSummary keeps its same-hub-file claim and cites that row.
- **F2** (card 16): creates `configengine.BelowLayer(dirs, spec, layer)`, the unexpanded merge of the template and every present file layer below `layer` through `Layers`.
  `ReconcileAll` strips against `BelowLayer(dirs, m.Spec(), LayerWorktree)` and reads the repo default for `Inherited` through `Layers`.
  `Uses:` drops `yamlengine.Merge` and adds `configengine.Layers`; `Edit:` adds `TestLayers`, which gains a `BelowLayer` row.
- **F3** (card 16): dropped `TestEnsure_MigratesEveryWorktreeToSparseOnce` and the hubreconcile budget rise.
  `Edit:` adds `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, `TestEnsure_MidMergePairIsLeftAloneAndRetried` and `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`, each with the assertions the finding lists.
- **F4** (card 7): the retired-key error names the layer file carrying the key (the highest `Layers` file holding `profiles`), with `lyx config reconcile --apply` for a worktree or local file and `run "lyx fabric reconcile"` plus the FAILED/orch clause for the repo default.
  `Edit:` adds `batcher#retiredKeyError` with its re-sign arrow, `loomshed#retiredKeyWayForward`, both `doc.go` files and `contracts/specs/refusal-spec.md` (rows "config file content invalid" and "Loom-Preflight batcher"); `Uses:` adds `configengine.Layer` and `Layers`.
  `TestActive` gains the repo-default retired-key row.
- **F5** (cards 4, 5, 9, 10): cards 4 and 5 drop `TestLoadConfig_ResolvesTheHubStack` and their budget rises (and the budget file from `Edit:`), putting one repo-default row and one local-refusal row into each package's `TestLoadConfig_TemplateDefaultsResolve`.
  Card 9 keeps only loom `parent_review_wait_min`; card 10 keeps only darn `verify` and landing `publish_verify`; the rest stay with card 13's `TestModules_LocalLayerRefusals`.
- **F6** (card 15): creates `configsync.HubWideOptions{Apply, SeedDefaults}` and re-signs `ReconcileHubWideAt(boardDir, primeBaseDir string, opts HubWideOptions)`.
  A present per-worktree default is always reconciled; an absent one (seed-only included) is seeded only under `SeedDefaults`.
  `hubreconcile.reconcileHubWide` (added to `Edit:`) passes true; `fabriccli.runReconcile` and `CloneAndWire` pass false.
  The Intent states this keeps D6's seed inside `hubreconcile.Ensure`, and `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains the flag-off row.
  `TestReconcileHubWideAt_MigratesLegacyFabricConfig`, a further caller of the re-signed function, is added to `Edit:`.
- **F7** (card 20): `outputSuccessWithTask` carries the arrow `func outputSuccessWithTask(out io.Writer, task boardengine.Task, notice string) int`, an empty notice omitting the field.
- **F8** (card 21): `stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer` drives a second task whose `board.json` entry carries `{"darn": {"verify": "x"}}` written straight into the board file, asserting Stuck naming `darn`, `verify` and the way forward, with no child seed and no local file.
- **F9** (cards 17, 20, 21): `TestDispatchLayers` gains a `--layer bogus` refusal row; `stepRunSettings` gains a `"run_settings": 3` refusal naming `run_settings`; `TestSeedChild_RunSettings` gains a failing-`ReadRunSettings` Stuck row with no seed written.
- **F10** (cards 15, 16): card 15's Intent states `configsync.go`'s new file header, the file staying out of `Edit:`; card 16's hubreconcile `doc.go` sentence says the walk's board commit carries the per-worktree repo defaults beside the hub-wide files.

## Gate repair

- **caller-uncovered, card 7** (`internal/batcher/registry.go:26`, `internal/batcher/estimate.go:173`): the re-signed `retiredKeyError` is called from `newCostFromProfile` and `profileWeights`, which hold a `profile` but not its file or layer.
  Card 7's `Edit:` adds `internal/batcher#profile`, `#newCostFromProfile` and `#profileWeights`; its Intent says `loadConfig` records the carrying file and layer on each decoded `profile` in two unexported, untagged fields, and the two sites keep their signatures and pass them on.
- **caller-uncovered, card 20** (`internal/boardcli/cli.go:143`): `outputSuccessWithTask` is also called from `Command`'s promote verb.
  Card 20's `Edit:` adds `internal/boardcli#Command`, and its Intent says promote passes an empty notice.
  The other two call sites sit in `storeVerbs`, which card 20 already lists.
- **create-new-unit, card 13** (informational): `pattern/PATTERN-config-layers.md` is the intended new background file for the new PATTERN entry; nothing changed.

## Disputed

None.

## Deferred

None.

## Note for the next round

With F6's flag, `lyx fabric reconcile` no longer seeds an absent per-worktree repo default.
Card 2 names `lyx fabric reconcile` as the way forward for any absent base under the board dir, so for a per-worktree default deleted after the walk has run (stamp current), that way forward would not restore it.
Before the first walk after upgrade, the start verbs run `hubreconcile.Ensure` and seed it, so only that post-migration case is affected.
I left card 2's message as it stands, because the fix turns on whether repairing a default after migration counts as D6 seeding, and the record does not settle that.
