# Card 4 — fabricengine-git-queries

**Edit:**
- `fabricengine/add_remoteleftover_integration_test.go`
- `fabricengine/add_rollback_adopt_test.go`
- `fabricengine/add_runrecords_integration_test.go`
- `fabricengine/archive_integration_test.go`
- `fabricengine/bolt_integration_test.go`
- `fabricengine/checkout_index_refresh_test.go`
- `fabricengine/checkout_rollback_test.go`
- `fabricengine/cleanup_primary_integration_test.go`
- `fabricengine/cleanupremote_integration_test.go`
- `fabricengine/clone_adopt_test.go`
- `fabricengine/commit_lock_integration_test.go`
- `fabricengine/config_driven_junctions_integration_test.go`
- `fabricengine/destroyremote_integration_test.go`
- `fabricengine/destructivegaps_integration_test.go`
- `fabricengine/dotlyxjunction_integration_test.go`
- `fabricengine/gitsha_integration_test.go`
- `fabricengine/headsha_integration_test.go`
- `fabricengine/healthreason_integration_test.go`
- `fabricengine/hook_test.go`
- `fabricengine/hubcontainment_integration_test.go`
- `fabricengine/hubscratch_integration_test.go`
- `fabricengine/junction_pattern_integration_test.go`
- `fabricengine/livestate_helpers_test.go`
- `fabricengine/livestate_refusal_selftest_test.go`
- `fabricengine/livestate_verbs_test.go`
- `fabricengine/merge_target_integration_test.go`
- `fabricengine/mergecrucible_integration_test.go`
- `fabricengine/mergein_integration_test.go`
- `fabricengine/mergein_recovery_integration_test.go`
- `fabricengine/mergesiblings_integration_test.go`
- `fabricengine/mergestage_integration_test.go`
- `fabricengine/mergeweftlocal_integration_test.go`
- `fabricengine/origin_integration_test.go`
- `fabricengine/pairbranch_archive_integration_test.go`
- `fabricengine/pull_integration_test.go`
- `fabricengine/reconcile_stale_registration_test.go`
- `fabricengine/remove_archive_integration_test.go`
- `fabricengine/remove_refusal_remedy_integration_test.go`
- `fabricengine/remove_warpbranch_integration_test.go`
- `fabricengine/removeremote_integration_test.go`
- `fabricengine/warpforward_integration_test.go`
- `fabricengine/weftgit_exclude_test.go`
- `fabricengine/weftgit_pathspec_integration_test.go`

**Uses:**
- `gitkit/query.go`

**Intent:**
Replace fabricengine's private git query helpers with the `gitkit` ones (#325 §1), so a query helper exists once instead of once per test package and file.

Delete each private query helper and point its callers at `gitkit`:

- ref → SHA: `shaOf`, `archiveTipOf`, `weftBareTip`, `branchTip`, `currentSHAOf`, `currentSHA`, `bareBranchSHA`, `mustGitHeadSHA`, `liveStateCurrentSHA`, `gitRevParse`, `resolveSHAForTest`, `rootCommitForTest` → `gitkit.RevParse` (or `gitkit.Git` for a `rev-list --max-parents=0` root lookup).
- current branch: `currentBranchOf`, `currentBranch`, `currentBranchName`, `readBranchForTest` → `gitkit.CurrentBranch`.
- branch exists: `branchExists`, `branchExistsLocally`, `branchExistsAt` → `gitkit.BranchExists`.
- ancestry: `isAncestorInCheckout`, `isAncestorForTest`, `gitIsAncestor` → `gitkit.IsAncestor`.
- counts: `gitCommitCount`, `gitRevListCount`, `revListCountBetween` → `gitkit.RevListCount`.
- tracked files: `gitLsFiles`, `lsFilesWeft`, `trackedUnderRoot` → `gitkit.LsFiles`.
- exclude file: `readExcludeLines`, `readWeftExcludeLines`, `excludeFilePath` → `gitkit.ExcludeLines`.

A helper whose signature takes `testing.TB` from a harness that has none (the `livestate_*` helpers taking `tb`) passes its `tb` through.
A helper that returns an error for the caller to assert rather than fataling (`readBranchForTest` in a health-reason test) stays only if its error is itself the asserted behaviour; otherwise it converts.
Every call site's assertion stays as it is; only the probe changes.
The `livestate_*` harness and its selftests are D9-protected: restructure only, every assertion survives.
The root-cause comment in `origin_integration_test.go` that explains the duplicate copy goes with the copy.
Every file here is already integration-tagged, so the D4 spawn rule holds.

Drop doc comments that restate a touched test's name, and comments that count rows or list seams (D10).

**ImpactSummary:** Integration-tagged fabricengine tests only; private query helpers deleted, probes routed through gitkit, no assertion changed.
