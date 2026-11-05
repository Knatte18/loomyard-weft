```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code and tests, through the fix-code-correctness lens.

**F1 (`originreconcile.go`).** `detachedBranch` is the constant the parser writes, so the comparison is correct. No finding.

**Findings**

1. **MEDIUM — `refuseWhenNothingRemains` / `refuseRemoval` (`remove.go`): a launcher-only pair now costs a network call.**
   - Input: a pair whose only local remnant is its launcher directory, in a repository with an origin.
   - Before the fix, `localPartRemains()` was true, so the function returned `"", nil` with no origin call.
   - Now `branchPartRemains()` is false, so `originPairBranch` runs, which calls `RemoteURL` and `remoteHeadTip` (an `ls-remote`).
   - This reaches `RemoveLocal`, because `refuseRemoval` runs in `localMode` too. The `RemoveLocal` doc says "It runs no ls-remote".
   - A failing lookup on a configured origin is returned from `refuseWhenNothingRemains`. A launcher-only `Remove` or `RemoveLocal` that used to succeed offline now errors.
   - The "nothing left" case already did this. The fix widens it to a case that was network-free, and `RemoveLocal`'s doc and tests are unchanged.

2. **LOW — `TaskWorktreeOnAnEarlierPrefixBranchIsRead` row (`pairbranch_integration_test.go`): it does not pin the task-step change alone.**
   - The row renames both branches and leaves both worktrees standing.
   - With the new `weftBranchExists(RecordsBranchName(branch))` clause in the task step reverted, the records-worktree step still returns `earlier/pt-pair`, because the warp branch exists locally. The row still passes.
   - The `RecordsWorktreeOnAnEarlierPrefixBranchIsRead` row does isolate the records step: with `exists` removed it falls through to `hanf/pt-pair` and fails.
   - Nothing asserts the task step on its own. The row would need the records worktree removed, or a records branch that does not map back to the warp branch.

3. **LOW — task-step acceptance by records-branch existence (`pairbranch.go`, `branch != "" && (... || weftBranchExists(...))`).**
   - Input: the task worktree of slug `a` is checked out onto another pair's live branch `b`. `b-weft` exists, so the step returns `b`.
   - `Remove(a)` would then target `b` and `b-weft`.
   - Git refuses the same branch in two worktrees, which narrows this to a detached or older `b`. The destructive gates also still apply.
   - It is the parent's accepted rule, but no test pins the negative with a branch that has a records branch. The existing unrelated-branch row uses `elsewhere`, which has none.

**Checked and holding**
- F3 adoption: `onOrigin != ""` is only returned when `branchPartRemains()` is false, so `pair.warpBranch = onOrigin` is correct without a `warpBranchFound` field. `localPartRemains` has no remaining callers in the context.
- `OnlyTheLauncherDirectoryLeft` fails with the defect in place: the remote job would target `hanf/pn-launchers` and the bare-slug branches would stay on origin.
- `UnresolvableRecordsRepositoryLeavesTheLocalBranchStep`: deleting the `backend` anchor directory makes `PrimeName`'s `List` fail, so the guard is exercised. It returns `slug` through the local-branch loop, and it fails if the `rootErr == nil` guard is removed.
- Records step: the new `localBranchExists` error return replaces a silent skip. That is a behaviour change, but it only fires when the task repository's branch lookup fails.

Further tool calls: none. My context already held every changed file and the review.
