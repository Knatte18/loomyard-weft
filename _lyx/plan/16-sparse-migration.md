# Card 16 — sparse migration

**Create:**
- `plan:internal/configengine#SparseMarker` -> `func SparseMarker(baseDir string) string`
- `plan:internal/configengine#SparseMarkerRel` -> `func SparseMarkerRel() string`
- `plan:internal/configengine#BelowLayer` -> `func BelowLayer(dirs Dirs, spec Spec, layer Layer) ([]byte, error)`
- `plan:internal/configsync#TestReconcileAll_MigratesToSparse` -> `func TestReconcileAll_MigratesToSparse(t *testing.T)`

**Edit:**
- `internal/configengine_test#TestConfigPaths`
- `internal/configengine_test#TestLayers`
- `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`
- `internal/hubreconcile_test#runStaleBuildReconcile`
- `internal/hubreconcile_test#TestEnsure_MidMergePairIsLeftAloneAndRetried`
- `internal/hubreconcile_test#TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`
- `internal/hubreconcile_test#newStaleHub`
- `internal/hubreconcile_test#seedRetiredKey`
- `internal/hubforge/doc.go`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli_test#TestRunCLI_CleanHubScenario`
- `plan:internal/configsync#Options`
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll`
- `internal/hubreconcile#reconcileWorktree`
- `internal/hubreconcile#snapshotConfig`
- `internal/hubreconcile#logResult`
- `internal/hubreconcile/doc.go`
- `internal/configcli#runReconcile`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/yamlengine#Strip`
- `internal/yamlengine#LeafPaths`
- `internal/configengine#Layers`

**Intent:**
Run D6's one-time migration of today's full worktree files to sparse overrides, inside the hub walk and pair call only (`PATTERN-binary-change-reconcile`).

`configengine.SparseMarker` returns the tracked marker `<baseDir>/_lyx/config/sparse` (no `.yaml` suffix, so no module reader takes it for a module file) and `SparseMarkerRel` its anchor-relative form; configengine is the only package forming the path (`PATTERN-config-layers`).
`configengine.BelowLayer` returns the unexpanded merge of the template and every present file layer below `layer`, read through `Layers` and merged as `Load` merges them, so the strip compares against the same stack the loaders resolve and configsync reads no layer file itself.
`configsync.Options` gains `Migrate bool`, and `Result` gains `Stripped` and `Inherited`, sorted dotted key lists.

When `boardDir` is set and the worktree lacks the marker, `ReconcileAll` computes, per per-worktree module whose worktree file is present, the strip against what the layers below resolve to, `configengine.BelowLayer(dirs, m.Spec(), configengine.LayerWorktree)`, compared with `yamlengine.Strip` under the module's open maps and `WholeEntries`, so a `${env:...}` marker equals only the same marker text in either direction, an open map or a `models` alias is kept whole unless equal as a unit, and the strip never changes what a key the file holds resolves to.
`Inherited` lists every unit the repo default, read as its `configengine.Layers` entry, holds that the worktree file lacked and whose value differs from the template's or that the template lacks, such as a seed-only alias, lens or fan the prime holds, so the operator can pin the old value with `lyx config <module> --set` in that worktree.
Seed-only modules take the strip too, the one rewrite D2 allows them.
With `Migrate` and `Apply` both set, the stripped files are written, a file left empty is deleted, and the marker is written after them; the strip runs only when every per-worktree module's repo default exists, so a walk whose hub-wide step was skipped migrates nothing and leaves the marker absent.
Without `Migrate`, as from `lyx config reconcile`, `Stripped` and `Inherited` are reported as pending and nothing is stripped, even under `--apply`.
A worktree carrying the marker is never stripped; a pair forked from a migrated worktree inherits the marker, and one forked before, or skipped as being created or mid-merge, or left by a walk that died partway, is stripped by the next walk.

`hubreconcile.reconcileWorktree` passes `Migrate: true`, commits the stripped and deleted files and the marker in the worktree's one existing commit, and `logResult` logs `stripped` and `inherited`.
`snapshotConfig` records `configengine.SparseMarker(anchor)` with its bytes when present and as absent (nil) when not, so `restoreConfig` removes a marker a failed or merge-interrupted walk wrote and the worktree stays unmarked for the next walk (D6).
`configcli.runReconcile` adds `stripped` and `inherited` to a module result when non-empty.

A clone's own `Ensure` walk, which card 15 has `CloneAndWire` end with, migrates the prime too: its repo defaults were seeded by the same walk, so the prime arrives with the sparse marker committed, and a pair forked from it inherits the marker.
`CloneAndWire` appends a `KindFileWritten` entry for `configengine.SparseMarker` of the prime when the marker is absent before the `Ensure` call and present after it, so the clone's mutation record covers it.
`internal/hubforge/doc.go`'s seeding contract says a clone's prime arrives with the sparse marker committed by the clone's walk.
`TestRunCLI_CleanHubScenario` asserts the prime's committed marker and the record's entry for it, and its `CloneConfigCommit_PairInheritsConfigs` step asserts the pair forked from the clone inherits the marker.
`doc.go` of hubreconcile says the walk's board commit carries the per-worktree repo defaults beside the hub-wide files, and describes the migration, the marker, the inherited report, the deletion of an emptied file and the R2 landing warning a migrated pair may see once.

`TestReconcileAll_MigratesToSparse` is table-driven over a temp board dir and anchor, comparing each module's resolution before and after through `configengine.Load` and `LoadOrTemplate`: keys equal to the default are stripped and reported, an emptied file is deleted, an open map differing from the default's in one entry is kept whole, a literal equal to the expansion of a marker below is kept, a marker over a literal equal to its expansion is kept, a seed-only key the default holds and the file lacks is reported inherited, every key the file held resolves unchanged, a marked worktree is left alone, and a dry run strips nothing.
`TestLayers` gains a `BelowLayer` row: below `LayerWorktree` in a hub it returns the template merged under the repo default, unexpanded, and ignores the worktree and local files.
Since every hubforge worktree now arrives marked and holding no per-worktree file, `newStaleHub` gives the walk unmarked worktrees with something to strip: after `CopyHub` and `AddPair` it writes and commits, in the prime and in the pair, a full pre-migration per-worktree file set, each module's template plus one key holding a value that differs from the repo default, and deletes and commits `configengine.SparseMarkerRel()` in each.
`newStaleHub` writes that full file set first and calls `seedRetiredKey` after it.
`seedRetiredKey` then commits `batcher.yaml` as the template with the retired key in place of `orientation`, plus the same one key whose value differs from the repo default.
So `batcher.yaml` still carries the retired key until a walk runs, and survives the strip non-empty, which keeps `retiredKeyPresent`'s read working after the walk.
The callers of `seedRetiredKey` that build on a marked pair, such as `TestEnsure_PairRegisteredDuringTheWalkIsReconciledBeforeTheStamp`, keep passing.
`TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` keeps building its hub through `newStaleHub`, so its no-marker-after-restore assertion starts from an unmarked pair.
Its hook keeps the clear func that `markMergeRecord` returns;
the test calls it after the restore assertions, then runs `Ensure` again and asserts the pair is stripped and committed with the sparse marker.
The walk's existing integration tests gain the migration's assertions: `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` asserts the defaults are seeded from the prime and committed on the board, each worktree is stripped and committed with its marker, and a second walk strips nothing: `runStaleBuildReconcile` removes the stamp at `geom.StampPath()` before its second `Ensure`, so the walk runs, then asserts it lands no new prime, pair or board commit and leaves each worktree's per-worktree files and sparse marker byte-identical.
Its case from card 15 that deletes loom's repo default reseeds it from the prime's `loom.yaml`, which `newStaleHub` made full, so the prime's and the pair's `loom.yaml` equal the reseeded default and the strip deletes them: the case asserts both files are absent after the walk and that each deletion is committed in that worktree's one reconcile commit.
`TestEnsure_MidMergePairIsLeftAloneAndRetried` asserts the skipped pair is stripped and marked by the retry; `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` asserts a worktree failed through `beforeCommit` has no marker on disk after the restore and, once the merge record is cleared, the next walk strips it.
`TestConfigPaths` pins the marker paths.
The budget row for `internal/configsync` rises by one.

**ImpactSummary:** adds `configengine.BelowLayer` and extends `configsync.Options` and `Result` and the walk's worktree commit; `lyx config reconcile` gains two report fields and never strips; a clone's prime arrives migrated, and the walk's fixture hub is unmigrated by `newStaleHub`.
