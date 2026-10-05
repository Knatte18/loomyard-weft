# Card 6 — Run fabricengine integration tests in parallel

**Edit:**
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/bolt_integration_test.go`
- `internal/fabricengine/checkout_rollback_test.go`
- `internal/fabricengine/clone_adopt_test.go`
- `internal/fabricengine/clone_explicit_root_subpath_test.go`
- `internal/fabricengine/clone_hubscratch_integration_test.go`
- `internal/fabricengine/coalesce_integration_test.go`
- `internal/fabricengine/commit_gating_integration_test.go`
- `internal/fabricengine/commit_integration_test.go`
- `internal/fabricengine/commit_lock_integration_test.go`
- `internal/fabricengine/commit_partial_integration_test.go`
- `internal/fabricengine/committed_lyxonly_integration_test.go`
- `internal/fabricengine/destroy_landedwork_integration_test.go`
- `internal/fabricengine/diff_integration_test.go`
- `internal/fabricengine/dotlyxjunction_integration_test.go`
- `internal/fabricengine/gitexclude_integration_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/hubscratch_integration_test.go`
- `internal/fabricengine/livestate_mutationoracle_selftest_test.go`
- `internal/fabricengine/livestate_verbs_selftest_test.go`
- `internal/fabricengine/mergecrucible_integration_test.go`
- `internal/fabricengine/mergein_integration_test.go`
- `internal/fabricengine/mergein_recovery_integration_test.go`
- `internal/fabricengine/mergelock_integration_test.go`
- `internal/fabricengine/mergesiblings_integration_test.go`
- `internal/fabricengine/mergestage_integration_test.go`
- `internal/fabricengine/mergestagetracked_integration_test.go`
- `internal/fabricengine/mergestateactive_integration_test.go`
- `internal/fabricengine/mergestate_integration_test.go`
- `internal/fabricengine/merge_target_integration_test.go`
- `internal/fabricengine/mergeweftlocal_integration_test.go`
- `internal/fabricengine/midmerge_integration_test.go`
- `internal/fabricengine/open_integration_test.go`
- `internal/fabricengine/openparent_integration_test.go`
- `internal/fabricengine/origin_integration_test.go`
- `internal/fabricengine/pathtracked_integration_test.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabricengine/pushrebasefree_integration_test.go`
- `internal/fabricengine/ready_integration_test.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/remove_junctions_integration_test.go`
- `internal/fabricengine/shortnamebinding_clone_integration_test.go`
- `internal/fabricengine/snapshot_integration_test.go`
- `internal/fabricengine/stencilcommit_integration_test.go`
- `internal/fabricengine/stencilhistory_integration_test.go`
- `internal/fabricengine/syncweft_integration_test.go`
- `internal/fabricengine/unwire_test.go`
- `internal/fabricengine/warpbinding_clone_integration_test.go`
- `internal/fabricengine/weftgit_exclude_test.go`
- `internal/fabricengine/weftgit_unborn_warp_test.go`
- `internal/fabricengine/weftguards_integration_test.go`

**Intent:**
Every top-level test in fabricengine's integration-tagged files that is still serial gains `t.Parallel` unless it touches process-global state (D5, `PATTERN-test-parallel`).
Each test builds its own hub or scratch repository, so independence is the expected case;
check each one for process-global state before adding the call: `t.Setenv` (which panics under `t.Parallel`), `os.Chdir`, a package-level variable it assigns, or a shared fixture directory.
A test that stays serial gets a comment at the test naming that state, or one comment atop its file when the whole file shares it (for example `coalesce_integration_test.go`'s `os.Chdir` and `hubcontainment_integration_test.go`'s and `checkout_rollback_test.go`'s `WEFT_SKIP_PUSH`).
Subtests are left as they are unless a table loop already shares nothing, in which case its subtests may call `t.Parallel` too, with the loop variable captured per iteration where the Go version needs it.
No assertion changes.
Run `go test -tags integration -race -count=3 ./internal/fabricengine/` before committing, to catch a test that shares state it does not name.

**ImpactSummary:** test-only, confined to `internal/fabricengine`'s integration-tagged test files; production code and other packages are untouched.
