```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (lens: code-tests)**

1. **Remove deletes a directory refilled after the sibling teardown** (`remove.go:327-335`).
   - Pinned by `TestRemove_DeletesTaskWorktreeDirRefilledAfterItsRemoval`. It checks the directory is gone, `StrayPath` is empty and a `KindPathRemoved` mutation is recorded.
2. **Remove reports `StrayPath` when the refilled directory cannot be deleted** (`refilledKept`, `remove.go:331-334` and `358`).
   - Pinned by none. The only refill test succeeds at the delete, so the `leftoverErr` branch and the `refilledKept → StrayPath` line could break unseen.
3. **Remove deletes a refill only when no registration covers the path** (`!isRegisteredLinkedWorktree`, `remove.go:330`).
   - Pinned by none. If the guard were dropped, the refill test would still pass, and the focus's question about a live registered worktree has no row.
4. **`batten cancel --remove` warns of a `StrayPath`** (`cancel.go:670`).
   - Pinned by the `RemoveLeavingAStrayDirectoryWarnsOfIt` row in `TestCancelRun`. It asserts the exact warning text and `PairRemoved`.
5. **Worktree-Create halts on `ErrTaskPathTaken` with a path and a way forward** (`wire.go:364`).
   - Pinned by the `TaskPathTaken` row in `TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout`.
6. **Prune reports a hub directory named as a pair slug as `Unowned`, and removes nothing** (`prune.go:183-202`).
   - Pinned by `TestPrune_ReportsANonWorktreeDirectoryAtAPairPath`, which runs dry-run, apply and apply+force.
7. **Prune's skip rules** (`prune.go:186-191`): a registered live worktree, a dot-prefixed directory, and a name `validateWorktreeSlug` rejects (the weft suffix and reserved names) are not reported.
   - Pinned by none. My grep for existing prune tests asserting empty entries returned nothing.
   - The only test is the positive one with a single stray directory, so a broken skip would go unnoticed. A broken `liveWarpSlugs` skip would report every real pair as `Unowned`, and the stray-directory test would still pass.
8. **`PairWorktreeStands` reads true only for a registered worktree** (`remove.go:1001`).
   - Pinned by `TestPairWorktreeStands_ReadsOnlyARegisteredWorktree`, with registered, plain-directory and absent rows.
   - The helper has no row for a registration whose directory is gone, which its doc comment claims reads false. `pathPresent` guards this.
9. **`board rename` and project change refuse a real pair and ignore a leftover directory** (`boardcli/cli.go:148`).
   - The refusal for a real pair is pinned by `TestCLI/Projects` (`verbs_test.go:519-530`), which now builds the pair with `git worktree add`.
   - Pinned by none for the leftover-directory side. No `boardcli` row makes a plain directory at the pair's path and expects a rename to succeed.
   - Reverting the seam to `os.Stat` would still pass `TestCLI/Projects`, because a real worktree is also a directory. Only the `fabricengine` function is pinned, not the wiring.
10. **`ErrTaskPathTaken` message wording** (`addadoptonly.go:76`).
    - Updated assertion in `TestAdd_LeftoverWorktreeDirErrorNamesRemedy`. This is static prose, so it is exempt.

**Findings**

- **F-A (MEDIUM, scope).** Location: `internal/fabricengine/remove.go:330-334` and `:358`; evidence: `remove_leftoverdir_integration_test.go:166-208`.
  - The new deletion has no row for a delete that fails, which should return success with `StrayPath` set.
  - It also has no row for a registered worktree at the path surviving, which the focus asks about.
  - Add both as rows or tests beside `TestRemove_DeletesTaskWorktreeDirRefilledAfterItsRemoval`. The failure case could use the `leftoverSeam.remove` seam to inject a non-late-writer error. The registered case needs a hook that re-registers a worktree at the path after the teardown.
- **F-B (MEDIUM, scope).** Location: `internal/fabricengine/prune.go:183-202`; evidence: `prune_unowned_integration_test.go:129-178`.
  - The skip rules have no negative rows: a live registered pair, a hidden directory, a `-weft` directory and a reserved name.
  - Add them as extra directories in the existing test, or as rows in a table, asserting no `Unowned` entry for them.
- **F-C (LOW, scope).** Location: `internal/boardcli/cli.go:148`; evidence: `verbs_test.go:519-530`.
  - No `boardcli` test shows that a plain directory at the pair's path no longer refuses a rename or project change.
  - Add a step to `stepProjects`, or a row, that creates a plain directory and expects success.

**Tool calls**
1. `grep` for prune tests asserting empty entries in `internal/fabricengine/*prune*_test.go`. My context held only the diff of the prune tests, not the other prune tests that could pin the live-pair skip.
