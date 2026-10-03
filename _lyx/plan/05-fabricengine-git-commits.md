# Card 5 — fabricengine-git-commits

**Edit:**
- `fabricengine/add_remoteleftover_integration_test.go`
- `fabricengine/add_rollback_adopt_test.go`
- `fabricengine/add_runrecords_integration_test.go`
- `fabricengine/clone_adopt_test.go`
- `fabricengine/clone_explicit_root_subpath_test.go`
- `fabricengine/clone_hubscratch_integration_test.go`
- `fabricengine/coalesce_integration_test.go`
- `fabricengine/destroy_landedwork_integration_test.go`
- `fabricengine/export_integration_test.go`
- `fabricengine/gitsha_integration_test.go`
- `fabricengine/headsha_integration_test.go`
- `fabricengine/index_integration_test.go`
- `fabricengine/livestate_manifest_selftest_test.go`
- `fabricengine/livestate_refusal_selftest_test.go`
- `fabricengine/livestate_states_test.go`
- `fabricengine/livestate_verbs_test.go`
- `fabricengine/merge_target_integration_test.go`
- `fabricengine/mergecrucible_integration_test.go`
- `fabricengine/mergein_integration_test.go`
- `fabricengine/mergein_recovery_integration_test.go`
- `fabricengine/mergelock_integration_test.go`
- `fabricengine/mergestage_integration_test.go`
- `fabricengine/mergestate_integration_test.go`
- `fabricengine/mergestateactive_integration_test.go`
- `fabricengine/mergeweftlocal_integration_test.go`
- `fabricengine/origin_integration_test.go`
- `fabricengine/pathtracked_integration_test.go`
- `fabricengine/prune_dirty_integration_test.go`
- `fabricengine/prune_unowned_integration_test.go`
- `fabricengine/pull_integration_test.go`
- `fabricengine/pushanchored_integration_test.go`
- `fabricengine/pushrebasefree_integration_test.go`
- `fabricengine/remove_archive_integration_test.go`
- `fabricengine/remove_guard_integration_test.go`
- `fabricengine/remove_refusal_remedy_integration_test.go`
- `fabricengine/remove_siblingdirty_integration_test.go`
- `fabricengine/remove_warpbranch_integration_test.go`
- `fabricengine/snapshot_integration_test.go`
- `fabricengine/warpbinding_clone_integration_test.go`
- `fabricengine/warpforward_integration_test.go`
- `fabricengine/weftguards_integration_test.go`

**Uses:**
- `gitkit/query.go`

**Intent:**
Replace fabricengine's private write-and-commit helpers and its inline add/commit pairs with `gitkit.CommitFile`/`gitkit.CommitFileOnBranch` (#325 §1).

Delete `commitFile`, `commitOnBranch`, `commitOnCurrentBranch`, `commitWarp`, `commitInWeft`, `commitPlain`, `commitFileOnBranch`, `landedWorkCommitFile`, `commitFileForPathTrackedTest` and `commitLyxFileInWeft`, routing each caller through the gitkit helper; a caller that used the returned SHA keeps using it.
A helper that adds more than a commit (a weft path computed from a fixture, a junction-relative file) shrinks to that path computation plus a gitkit call, or inlines at its sole caller.
Replace each inline `MustRun(... "git", "add" ...)` / `MustRun(... "git", "commit" ...)` pair that commits a file just written with one `gitkit.CommitFile`; a pair staging several paths or using flags `CommitFile` lacks (`--allow-empty`, `-a`) becomes `gitkit.Git` calls rather than a new private helper.
Repo-initialisation sequences (`git init`, `git config user.*`) go through `gitkit.Git` too.

Assertions are unchanged.
The `livestate_*` harness and its selftests are D9-protected: restructure only.
Every file here is integration-tagged, so the D4 spawn rule holds.
Apply D10 to every touched test.

**ImpactSummary:** Integration-tagged fabricengine tests only; private commit helpers and inline add/commit pairs collapse onto gitkit, no assertion changed.
