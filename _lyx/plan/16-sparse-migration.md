# Card 16 — sparse migration

**Create:**
- `plan:internal/configengine#SparseMarker` -> `func SparseMarker(baseDir string) string`
- `plan:internal/configengine#SparseMarkerRel` -> `func SparseMarkerRel() string`
- `plan:internal/configsync#TestReconcileAll_MigratesToSparse` -> `func TestReconcileAll_MigratesToSparse(t *testing.T)`
- `plan:internal/hubreconcile_test#TestEnsure_MigratesEveryWorktreeToSparseOnce` -> `func TestEnsure_MigratesEveryWorktreeToSparseOnce(t *testing.T)`

**Edit:**
- `internal/configengine_test#TestConfigPaths`
- `plan:internal/configsync#Options`
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll`
- `internal/hubreconcile#reconcileWorktree`
- `internal/hubreconcile#logResult`
- `internal/hubreconcile/doc.go`
- `internal/configcli#runReconcile`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/yamlengine#Strip`
- `plan:internal/yamlengine#Merge`
- `plan:internal/yamlengine#LeafPaths`

**Intent:**
Run D6's one-time migration of today's full worktree files to sparse overrides, inside the hub walk and pair call only (`PATTERN-binary-change-reconcile`).

`configengine.SparseMarker` returns the tracked marker `<baseDir>/_lyx/config/sparse` (no `.yaml` suffix, so no module reader takes it for a module file) and `SparseMarkerRel` its anchor-relative form; configengine is the only package forming the path (`PATTERN-config-layers`).
`configsync.Options` gains `Migrate bool`, and `Result` gains `Stripped` and `Inherited`, sorted dotted key lists.

When `boardDir` is set and the worktree lacks the marker, `ReconcileAll` computes, per per-worktree module whose worktree file is present, the strip against what the layers below resolve to: the template filled under the repo default, merged unexpanded, compared with `yamlengine.Strip` under the module's open maps and `WholeEntries`, so a `${env:...}` marker equals only the same marker text in either direction, an open map or a `models` alias is kept whole unless equal as a unit, and the strip never changes what a key the file holds resolves to.
`Inherited` lists every unit the repo default holds that the worktree file lacked and whose value differs from the template's or that the template lacks, such as a seed-only alias, lens or fan the prime holds, so the operator can pin the old value with `lyx config <module> --set` in that worktree.
Seed-only modules take the strip too, the one rewrite D2 allows them.
With `Migrate` and `Apply` both set, the stripped files are written, a file left empty is deleted, and the marker is written after them; the strip runs only when every per-worktree module's repo default exists, so a walk whose hub-wide step was skipped migrates nothing and leaves the marker absent.
Without `Migrate`, as from `lyx config reconcile`, `Stripped` and `Inherited` are reported as pending and nothing is stripped, even under `--apply`.
A worktree carrying the marker is never stripped; a pair forked from a migrated worktree inherits the marker, and one forked before, or skipped as being created or mid-merge, or left by a walk that died partway, is stripped by the next walk.

`hubreconcile.reconcileWorktree` passes `Migrate: true`, commits the stripped and deleted files and the marker in the worktree's one existing commit, and `logResult` logs `stripped` and `inherited`.
`configcli.runReconcile` adds `stripped` and `inherited` to a module result when non-empty.
`doc.go` of hubreconcile describes the migration, the marker, the inherited report, the deletion of an emptied file and the R2 landing warning a migrated pair may see once.

`TestReconcileAll_MigratesToSparse` is table-driven over a temp board dir and anchor, comparing each module's resolution before and after through `configengine.Load` and `LoadOrTemplate`: keys equal to the default are stripped and reported, an emptied file is deleted, an open map differing from the default's in one entry is kept whole, a literal equal to the expansion of a marker below is kept, a marker over a literal equal to its expansion is kept, a seed-only key the default holds and the file lacks is reported inherited, every key the file held resolves unchanged, a marked worktree is left alone, and a dry run strips nothing.
`TestEnsure_MigratesEveryWorktreeToSparseOnce` drives the walk over a hubforge hub with a prime and a pair: the defaults are seeded from the prime and committed on the board, each worktree is stripped and committed with its marker, a pair skipped as mid-merge is stripped by the next walk, a walk failed through the existing `beforeCommit` hook after the board's commit leaves that worktree unmarked and the next walk strips it, and a second walk strips nothing; `TestConfigPaths` pins the marker paths.
The budget rows for `internal/configsync` and `internal/hubreconcile` rise by one each.

**ImpactSummary:** extends `configsync.Options` and `Result` and the walk's worktree commit; `lyx config reconcile` gains two report fields and never strips.
