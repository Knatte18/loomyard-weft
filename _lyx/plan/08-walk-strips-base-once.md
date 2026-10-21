# Card 8 — walk-strips-base-once

**Create:**
- `plan:internal/hubreconcile_test#commitPreMigrationBase` -> `func commitPreMigrationBase(t *testing.T, boardDir string)`
- `plan:internal/hubreconcile#EnsureFailingAfterHubWideWrite` -> `func EnsureFailingAfterHubWideWrite(geom Geometry, opts Options, failure error) error`

**Edit:**
- `internal/hubreconcile#ensure`
- `internal/hubreconcile#Ensure`
- `internal/hubreconcile#repoDefaultAbsent`
- `internal/hubreconcile#reconcileHubWide` -> `func reconcileHubWide(boardDir, primeAnchor, label string, hooks walkHooks) (skipped bool, err error)`
- `internal/hubreconcile#walkHub`
- `internal/hubreconcile#snapshotConfig`
- `internal/hubreconcile#walkHooks`
- `internal/hubreconcile#WorktreeError`
- `internal/hubreconcile#WorktreeError.Error`
- `internal/hubreconcile/doc.go`
- `internal/hubreconcile_test#commitPreMigrationFiles`
- `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`
- `internal/hubreconcile_test#runStaleBuildReconcile`
- `internal/hubreconcile_test#TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`
- `internal/hubreconcile_test#TestEnsure_MidMergePairIsLeftAloneAndRetried`
- `internal/hubreconcile_test#TestEnsure_PairCallReconcilesDespiteAFreshStamp`

**Uses:**
- `internal/configsync#ReconcileHubWideAt`
- `internal/configengine#BaseSparseMarker`
- `internal/configengine#BaseSparseMarkerRel`
- `internal/configengine#SparseBaseHeader`
- `internal/hubreconcile_test#newStaleHub`
- `internal/hubreconcile_test#geometryOf`
- `internal/hubreconcile_test#writeCurrentStamp`
- `internal/hubreconcile_test#stampKey`
- `internal/hubreconcile_test#runningKey`
- `internal/hubreconcile_test#commitCount`
- `internal/hubreconcile_test#assertClean`
- `internal/hubreconcile_test#markMergeRecord`

**Intent:**
The walk runs D3's one-time base strip and commits the board marker with it, and an absent marker makes the stamp stale.

`reconcileHubWide` calls `ReconcileHubWideAt` with `StripBase: true` beside `Apply` and `SeedDefaults`, so only the walk strips a present base file (`PATTERN-binary-change-reconcile`).
When `configengine.BaseSparseMarker(boardDir)` did not exist before the pass and does after it, its path, `configengine.BaseSparseMarkerRel()`, joins the written list, so the marker lands in the walk's one board commit beside the files.
`snapshotConfig` also snapshots `configengine.BaseSparseMarker(anchor)`, absent mapping to nil like the worktree `sparse` marker, so a failed hub-wide step, a reconcile, commit or pull-then-commit failure, restores the board's config files to their prior bytes and removes a marker the pass wrote, through the existing `failRestoringConfig`.
A failure in a later worktree step leaves the committed board and its marker in place, and the next walk strips no base again and retries the worktrees, whose former-value drop (card 5) runs whether or not the marker exists.

`walkHooks` gains `afterHubWideWrite`, a test seam run inside the board write after `ReconcileHubWideAt` returns, whose non-nil error fails the write as a commit failure would, after the files and the marker are written; `walkHub` passes its hooks to `reconcileHubWide`, which takes them as a new parameter.
`EnsureFailingAfterHubWideWrite`, in `export_test.go` beside `EnsureWithHooks`, runs the walk with that hook returning failure, so the external test package reaches the restore of a written marker.

`repoDefaultAbsent` keeps its name and also reports stale when the board marker is absent, so a stamp naming the running build is stale on an unmigrated board, and a marker removed by hand with a board commit makes the next start verb's walk strip the base files again; its comment says it covers the marker, and `ensure`'s fast-path comment and `Ensure`'s doc say a matching stamp costs one file read and one stat per per-worktree repo default plus one for the marker.
The config registry's fingerprint already covers the template text, so this change's template edits make every hub's stamp stale and its next start verb runs the walk.

`WorktreeError` gains a `Board bool` field, set by `reconcileHubWide` on the error it returns, and `WorktreeError.Error` names the board-dir way forward: run `lyx config <module> --layer default` from the hub's prime worktree for a board file, the form `reviewbudget.Route` and `configcli` already use for a board file, since a bare `lyx config <module>` addresses a per-worktree module's worktree file.
A `WorktreeError` whose `File` is set and whose `Module` is empty, the board marker's write failure (card 6), names only the file to fix and the re-run, never a `lyx config` command.

`internal/hubreconcile/doc.go`'s walk and sparse-migration sections state: the walk strips the board's base files once, while the board marker `<BoardDir>/_lyx/config/base-sparse` is absent, of units equal to the template or a former value, writes a key-less base as the header-only file, writes the marker only when every registered module's base exists, and commits files and marker in the board commit; a failed hub-wide step removes the marker the pass wrote; each unmarked worktree's strip also drops former values; the stamp is stale while the marker is absent.

The fixtures hold former values where the walk must drop them:
- `commitPreMigrationBase` writes, at the board dir, every non-seed-only module's base as its current template with every D1 unit at its former value: loom's lenses, models, fans, `driver`, `discussion_max_bounces`, `webster_max_bounces` and `webster_auto_accept`, shed's `driver_kind: llm`, darn's models and fans, claude's `prompt_cache_ttl_roles` map `{driver: 1h, webster: 1h}`, orch's 300000 and 400000, webster's `kickstart_pack: true` and batcher's `cautious` `budget: 450000`, plus darn's `verify` set to a hub's own command; it removes `BaseSparseMarkerRel` and commits it on the board, and fails the test when a line it rewrites is missing from a template.
- `commitPreMigrationFiles` also writes loom's `discussion_lenses: ""` and shed's `driver_kind: llm` into each unmarked worktree file, former values that differ from the new template, so only the former-value drop removes them; its existing `review_max_bounces: 99` line still survives the strip.

`TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` gains rows through new `runStaleBuildReconcile` parameters:
- an unmigrated board (`commitPreMigrationBase` before the walk): after `Ensure` every non-seed-only base file other than darn's holds only `configengine.SparseBaseHeader`, darn's keeps its `verify`, the board tracks the marker in the walk's commit, `loomengine.LoadConfig` over the prime resolves `DiscussionLenses` `twill-discussion` and a `WebsterMaxBounces` pointing at 1, and `models.yaml` is byte for byte as seeded; the helper's second walk, run after loom's base is set to `driver: sonnet[medium]` and committed, keeps that unit, since the marker is present;
- the marker removed by a board commit while the stamp names the running build: the next `Ensure` walks and strips again, so a base unit equal to the template committed after the first walk is gone.
On every row the prime's and pair's files lose `discussion_lenses` and `driver_kind`, and loom's keeps only the `review_max_bounces: 99` line.
`TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` gains two rows on an unmigrated board: a hub-wide reconcile failing on an unparseable base file, whose error names `lyx config <module> --layer default` and leaves the board's config files at their prior bytes; and `EnsureFailingAfterHubWideWrite`, whose failure comes after the files and the marker were written, which leaves the board's config files at their prior bytes and no marker.
`TestEnsure_MidMergePairIsLeftAloneAndRetried` asserts that the skipped pair keeps its `discussion_lenses: ""` after the migrating walk and drops it in the next walk, which runs with the board marker present.
`TestEnsure_PairCallReconcilesDespiteAFreshStamp` asserts that the pair call drops the pair's `discussion_lenses: ""`, on a board holding the marker.

**ImpactSummary:** the walk's hub-wide step, its snapshot, test seam and error text, and the stamp's staleness test in `hubreconcile`; every start verb's walk on a hub not yet migrated strips its base once, and every other `ReconcileHubWideAt` caller is untouched.
