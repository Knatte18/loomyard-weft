---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/02-configengine-layer-stack.md:85
    summary: Probing _lyx/ on Anchor makes every hub-wide load in cards 10-12 fail, and every in-hub per-worktree load ignore the repo default, when the reading worktree's _lyx/ is absent, which also breaks lyx fabric reconcile's repair of a missing _lyx side.
    fix: >-
      In card 2's Intent, replace "the `_lyx/` probe runs on `Anchor` when set, else on `Board`" with "the `_lyx/` probe runs on the base layer's directory, `Board` when set, else `Anchor`; an absent `_lyx/` under `Anchor` while `Board` is set means the worktree and local layers are absent, never an uninitialised base", and narrow LoadOrTemplate's degrade to "an absent base-layer `_lyx/` or base file". Add two rows to card 2's `TestLoad_LayerStack`, each with `Dirs{Anchor: <dir without _lyx/>, Board: <initialised board dir>}`. In the first, a hub-wide spec loads its hub file under strict `Load`. In the second, a per-worktree spec resolves a repo-default key under both `Load` and `LoadOrTemplate`. In card 12's ImpactSummary, keep the claim of reading the same hub file only once that row pins it.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/16-sparse-migration.md:31
    summary: Card 16 stacks the template and the repo default inside configsync (template filled under the repo default, merged unexpanded with yamlengine.Merge), against the record's PATTERN-config-layers constraint that only configengine reads or stacks layer files.
    fix: Card 16 creates `plan:internal/configengine#BelowLayer -> func BelowLayer(dirs Dirs, spec Spec, layer Layer) ([]byte, error)`, which returns the unexpanded merge of the template and every present file layer below `layer` through `Layers` and the same merge `Load` uses, with a row in `TestLayers` (card 2's test, edited by card 16) pinning it. `ReconcileAll` computes the strip against `configengine.BelowLayer(dirs, m.Spec(), configengine.LayerWorktree)` and reads no repo-default file itself. Drop `plan:internal/yamlengine#Merge` from card 16's `Uses:`, add `plan:internal/configengine#Layers`, and reword line 31 to name `BelowLayer` as the source of what the layers below resolve to.
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/plan/16-sparse-migration.md:43
    summary: Card 16 adds a new top-level TestEnsure_MigratesEveryWorktreeToSparseOnce, where the record's test constraint says hubreconcile scenarios extend their existing integration tests, which already cover each scenario it drives (PATTERN-test-economy).
    fix: >-
      Drop `plan:internal/hubreconcile_test#TestEnsure_MigratesEveryWorktreeToSparseOnce` from card 16's `Create:` and the hubreconcile budget rise. Add to card 16's `Edit:` `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, which gains these assertions: defaults seeded from the prime and committed on the board, each worktree stripped and committed with its marker, a second walk strips nothing. Also add `internal/hubreconcile_test#TestEnsure_MidMergePairIsLeftAloneAndRetried`, which gains: the skipped pair is stripped and marked by the retry. Also add `internal/hubreconcile_test#TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`, which gains: a worktree failed through `beforeCommit` stays unmarked and the next walk strips it. Reword line 43 to name those three tests.
  - id: F4
    severity: BLOCKING
    class: scope
    location: _lyx/plan/07-batcher-layered-load.md:64
    summary: Card 7 lets a retired batcher.yaml key in the repo default reach the retired-key refusal, whose way forward, lyx config reconcile --apply, never reconciles the default layer in a hub (card 15 confines ReconcileAll to the worktree and local layers), so the named way forward does not clear it (PATTERN-refusal-way-forward).
    fix: "In card 7, the retired-key error names the layer file that carries the key, and for a key in the repo default names `run \"lyx fabric reconcile\"` followed by the clause `; a session lyx refuses the verb from reports status: FAILED and the orch runs it` in place of `lyx config reconcile --apply`. Add `internal/batcher/config.go`'s retired-key error site as a member glyph, `internal/loomshed#retiredKeyWayForward`, `internal/batcher/doc.go`, `internal/loomshed/doc.go` and `contracts/specs/refusal-spec.md` (rows config file content invalid and Loom-Preflight batcher) to card 7's `Edit:`. Add a `TestActive` row in which a retired key in the repo default fails naming the default file and `lyx fabric reconcile`."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-reed-layered-load.md:70
    summary: Cards 4 and 5 add new top-level TestLoadConfig_ResolvesTheHubStack tests while cards 6-12 add rows to existing load tests, and the loader cards re-pin per-key local refusals that card 13's TestModules_LocalLayerRefusals already pins for the same keys.
    fix: In card 4, drop `TestLoadConfig_ResolvesTheHubStack` from `Create:` and its budget rise, and put its rows in `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`. In card 5, do the same into `internal/shuttleengine_test#TestLoadConfig_TemplateDefaultsResolve`. Each loader card keeps one repo-default row and one local-refusal row proving the loader passes its `ConfigSpec`. Cards 9 and 10 cut their per-key refusal lists to one key each (loom `parent_review_wait_min`, darn `verify`, landing `publish_verify`), leaving the remaining acceptance keys to card 13's `TestModules_LocalLayerRefusals`.
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/plan/15-configsync-layer-reconcile.md:41
    summary: Card 15 puts D6's seeding of absent per-worktree repo defaults into ReconcileHubWideAt, which lyx fabric reconcile and CloneAndWire also call, so part of the D6 migration runs outside hubreconcile.Ensure, against the record's PATTERN-binary-change-reconcile constraint, and the card never addresses that constraint.
    fix: In card 15, `ReconcileHubWideAt` reconciles a present per-worktree default always but seeds an absent one only when the caller passes a new `SeedDefaults bool` option (a field of an options struct for `ReconcileHubWideAt`, given its signature with the arrow on the Edit glyph). `hubreconcile`'s walk passes true. `fabriccli.runReconcile` and `CloneAndWire` pass false. Card 15's Intent states that this keeps the D6 seed inside `hubreconcile.Ensure`, and the `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` rows add the flag-off case leaving an absent default absent.
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/20-board-run-settings.md:20
    summary: Card 20's Intent makes outputSuccessWithTask carry a notice, but its Edit glyph has no re-sign arrow while the function today takes only (out io.Writer, task boardengine.Task).
    fix: Replace the line with `- internal/boardcli#outputSuccessWithTask -> func outputSuccessWithTask(out io.Writer, task boardengine.Task, notice string) int`, an empty notice omitting the field.
  - id: F8
    severity: MEDIUM
    class: scope
    location: _lyx/plan/21-seed-child-run-settings.md:39
    summary: Card 21's Stuck route for its named input, a stale board entry, is reached only through a stub PrepareRunSettings; the battencli wrap of configreg.RunSettingsError into ErrInvalidRunSettings, which decides Stuck rather than a hard error, is reached by no test.
    fix: "Extend `stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer` with a second task whose `board.json` entry carries `{\"darn\": {\"verify\": \"x\"}}`, written straight into the board file and bypassing `lyx board upsert`. Assert Seed-Child stands Stuck with a reason naming `darn`, `verify` and the way forward, and that no child seed and no local file are written."
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/plan/17-config-cli-layers.md:49
    summary: Three new guard branches have no named test. Card 17's refusal of a --layer value outside default, worktree and local. Card 20's refusal of a non-object run_settings, which is reachable because boardcli decodes the payload into map[string]any. Card 21's Stuck on an unreadable board in ReadRunSettings, which TestSeedChild_UnreadableBoardIsStuck does not reach since it fails ReadBoardType.
    fix: "Add a `--layer bogus` refusal row to card 17's `TestDispatchLayers`. Add a `\"run_settings\": 3` refusal row to card 20's `stepRunSettings`. Add a row to card 21's `TestSeedChild_RunSettings` in which `ReadRunSettings` fails and Seed-Child is Stuck with no seed written."
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/plan/15-configsync-layer-reconcile.md:49
    summary: The record's docs constraint names internal/configsync, but no card updates configsync.go's file header, which says ReconcileHubWideAt covers the hub-wide modules only. hubreconcile/doc.go's walk paragraph, which says the board commit carries the hub-wide config, is not updated for the per-worktree defaults either.
    fix: Card 15's Intent adds that `internal/configsync/configsync.go`'s file header says `ReconcileHubWideAt` covers every module's file at the board dir (hub files and per-worktree repo defaults) and `ReconcileAll` a worktree's sparse and local layers. The file stays out of `Edit:` beside its members, because `redundant-file-target` refuses that. Card 16's `doc.go` sentence adds that the walk's board commit carries the per-worktree repo defaults beside the hub-wide files.
---

### [BLOCKING] The `_lyx/` probe on `Anchor` breaks hub-wide and in-hub loads

**Location:** `_lyx/plan/02-configengine-layer-stack.md:85`

**Issue:** Today `load` runs `FindBaseDir(baseDir)` on the dir that holds the one file (`internal/configengine/config.go:130`).
For the hub-wide modules that dir is the board dir.
Card 2 moves the probe to `Anchor` whenever it is set.
Cards 10–12 then tell darn, landing, gate, board and fabric `fabricengine.ConfigDirs(location)`, so a strict hub-wide `Load` now returns `ErrNotInitialized` when the reading worktree's `_lyx/` is absent, even though the base layer, the hub file under `Board`, is present.
Card 12 routes `RepoWiredNames`, `junctionNames`, `commitPendingRecords`, `Topology.RemoveRefusal` and the fabriccli verbs through `ConfigDirs(l)`.
`lyx fabric reconcile`, which `refusal-spec.md`'s preflight row names as the repair that "recreates a missing _lyx worktree and re-points broken junctions", then cannot load fabric's config in the very state it repairs.
For per-worktree modules in a hub, a `LoadOrTemplate` with the anchor's `_lyx/` absent degrades to the template and drops a present repo default.
Card 12's ImpactSummary claims "all reading the same hub file as before", which this contradicts.

### [BLOCKING] The migration strip stacks layers outside configengine

**Location:** `_lyx/plan/16-sparse-migration.md:31`

**Issue:** The record's constraint for the new `PATTERN-config-layers` reads "resolving a module's config, only `internal/configengine` reads or stacks layer files".
Card 13 lands that entry before card 16.
Card 16 has `configsync.ReconcileAll` compute "what the layers below resolve to: the template filled under the repo default, merged unexpanded", with `yamlengine.Merge` in its `Uses:`.
That is a second layer stack, owned by configsync, that can drift from `Load`'s (for example on `WholeEntries` or open-map carry).

### [BLOCKING] A new hubreconcile integration test where the record says to extend the existing ones

**Location:** `_lyx/plan/16-sparse-migration.md:43`

**Issue:** The record's constraint names `PATTERN-test-speed` and `PATTERN-test-economy`: "hubreconcile and batten scenarios extend their existing integration tests".
`internal/hubreconcile/ensure_integration_test.go` already holds the scenarios card 16's new test re-drives.
`TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` covers a full walk committing every worktree and the board.
`TestEnsure_MidMergePairIsLeftAloneAndRetried` covers a skipped pair retried.
`TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` covers the `beforeCommit` failure.
Card 21 follows the constraint with a step under `TestBattenIntegration_Rows`, and card 16 does not.

### [BLOCKING] Retired-key way forward does not reach the repo default

**Location:** `_lyx/plan/07-batcher-layered-load.md:64`

**Issue:** Card 7 states that a retired `batcher.yaml` key in any layer reaches the retired-key refusal.
That refusal's way forward is `lyx config reconcile --apply` (`internal/batcher/config.go:46`, `loomshed.retiredKeyWayForward`, refusal-spec rows "config file content invalid" and "Loom-Preflight batcher").
`configcli.runReconcile` calls only `configsync.ReconcileAll` (`internal/configcli/configcli.go:321`), and card 15 limits that, in a hub, to the worktree and local layers.
The repo default is reconciled only by `ReconcileHubWideAt`, through the walk, `lyx fabric reconcile` or a clone.
A retired key in the default stays reachable: `validateHubFile`'s strict `Load` tolerates keys the template lacks, so a `--layer default --set` can write one.
The named way forward then leaves the refusal standing.

### [MEDIUM] Loader cards duplicate tests that already cover the behaviour

**Location:** `_lyx/plan/04-reed-layered-load.md:70`

**Issue:** Cards 4 and 5 each create a new top-level `TestLoadConfig_ResolvesTheHubStack`, while cards 6–12 put the same repo-default row into each module's existing load test.
reed and shuttle both have `TestLoadConfig_TemplateDefaultsResolve`, which can take the row.
The loader cards also pin specific local refusals: reed `shell`, shuttle `claude_deny_python`, burler `lenses`, loom `parent_review_wait_min`/`discussion_producer`/`plan_fan`, darn `verify`/`review_max_bounces`, landing `publish_verify`/`require_pr_to_base`.
Card 13's `TestModules_LocalLayerRefusals` pins each of those keys against the same `ConfigSpec`, so per-key coverage is doubled.
A loader needs only one refusal row to prove it passes its spec.

### [MEDIUM] D6's default seeding escapes `hubreconcile.Ensure`

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md:41`

**Issue:** `configsync.ReconcileHubWideAt` is called by `hubreconcile`'s walk (`internal/hubreconcile/walk.go:185`), by `lyx fabric reconcile` (`internal/fabriccli/fabric.go:649`) and by `CloneAndWire` (`internal/fabriccli/clone.go:54`).
Card 15 makes it seed every absent per-worktree repo default from the prime "(D6)".
The record's `PATTERN-binary-change-reconcile` constraint says "the layer reconcile and the D6 migration run only inside `hubreconcile.Ensure`'s walk and pair call".
The card neither confines the seed nor states why the other two callers fall outside the constraint.

### [MEDIUM] Missing re-sign arrow on `outputSuccessWithTask`

**Location:** `_lyx/plan/20-board-run-settings.md:20`

**Issue:** `internal/boardcli/cli.go:894` is `func outputSuccessWithTask(out io.Writer, task boardengine.Task) int`.
Card 20's Intent has it add a `notice` field it is handed.
That changes its parameters, so the Edit glyph needs the arrow.

### [MEDIUM] Seed-Child's Stuck route is tested only through a stub

**Location:** `_lyx/plan/21-seed-child-run-settings.md:39`

**Issue:** The card names the input that reaches the Stuck branch: a board entry whose `run_settings` no longer validate, or were written to `board.json` by hand.
`TestSeedChild_RunSettings` stubs `PrepareRunSettings` to return `ErrInvalidRunSettings`, so it never exercises battencli's wrap of `*configreg.RunSettingsError`.
`TestWire_EverySeamFilled` checks only that the seams are present.
If the wrap were missing, the named input would surface as a hard error instead of the D9 Stuck reason, and no named test would catch it.

### [LOW] Three guard branches with no named test

**Location:** `_lyx/plan/17-config-cli-layers.md:49`

**Issue:** Card 17's `dispatch` validates `--layer` against three names, and no row passes a fourth.
Card 20's `prepareRunSettings` refuses a non-object value. `boardcli` decodes upsert payloads into `map[string]any` (`internal/boardcli/cli.go:368`), so nothing upstream rejects `"run_settings": 3`, yet `stepRunSettings` has no such row.
Card 21 adds a Stuck branch for an unreadable board in `ReadRunSettings`, but the existing `TestSeedChild_UnreadableBoardIsStuck` fails `ReadBoardType` first, and the new table has no row for it.

### [LOW] Docs constraint misses configsync and the walk's board commit

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md:49`

**Issue:** `internal/configsync/configsync.go`'s file header (lines 5–6) says "ReconcileHubWideAt covers the hub-wide modules at the board dir", which card 15 makes false.
The record's docs constraint names `internal/configsync`.
`internal/hubreconcile/doc.go:17` says the walk "reconciles the hub-wide config and commits it on the board".
Card 15 adds the per-worktree defaults to that commit, but card 16's doc edit covers only the migration.

## Rejected

- Focus 1, fidelity: each of D1–D10, the six post-Discussion additions and every Constraint is carried by some card, apart from the gaps in F2, F3, F6 and F10. Cards 16 (D6), 18 (D7's `--resolved`) and 19 (D8's "same guard function") are each licensed by the record. Card 3's `ConfigDirs` and card 7's `shedrecipe.Env.ConfigDirs` are the told-geometry plumbing D1 and `PATTERN-told-geometry` need.
- Focus 2, card order: no card uses a symbol a later card creates. Cards 4–12 use only cards 2 and 3. Card 13 uses the specs from cards 4–12. Cards 14, 15 and 17 use `Module.Spec` from card 13. Card 18 uses card 17's `layerFile`. Card 20 uses cards 14 and 19, and card 21 uses card 14. configreg importing every engine creates no cycle: `go list -deps` shows that none of the fourteen engine packages reaches `internal/configreg`.
- Focus 3, attack surface: the Intent of cards 13, 14, 17, 20 and 21 each states what the new path lets through and its bound. Card 13 gives D4's category lists. Card 14 gives `ValidateLocal`. Card 17's `--layer local` is uncommitted and refuses D5 keys, and `--layer default` goes through the unchanged hub-wide guard. Card 20 gives the slug guard with its stated mistaken-not-determined bound. Card 21 gives D9's write-once bound. Not a defect.
- Focus 4, reachability: card 2's absent board-dir base row, card 14's refusals, card 17's `--layer worktree` refusal and card 20's guard and notice each have a named input and test. The gaps are F8 and F9.
- Focus 5, Edit glyphs in cards 4–12: each re-signed loader carries its arrow, including `DiscussionSpec`, `twillBlocks`, `validateFanKeys`, `VerifyCommand`, `WiredNames`, `PathspecNames`, `junctionNames` and the three loomshed constructors. Glyphs without arrows (`RepoWiredNames`, `OpenHub`, `GateSlots`, `darnVerifySource`, `wireDarn`, `commitPendingRecords`, `RemoveRefusal`) keep their current signatures, checked against the code. Not a defect.
- Focus 7, verify: the commands run in the order the plan template states, `go test ./...` and `-tags integration ./...` cover every targeted package, and `tools/` has no nested `go.mod`. Not a defect.
- The per-key classification table lists every top-level key of every template (checked against each `template*.yaml`). `loom.review_max_bounces` as none is correct, since its load rejects a value below 1 (`internal/loomengine/config.go:4`).
