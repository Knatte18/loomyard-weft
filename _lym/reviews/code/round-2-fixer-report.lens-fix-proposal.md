```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out (fix-proposal lens). F1 and F4 were fixed as proposed; F2 and F3 were fixed with the fixer's own change.

**MEDIUM — `pairTaskBranchLocal` task-worktree and records-worktree acceptance (`internal/fabricengine/pairbranch.go`).** The records-branch-exists test no longer rules out a branch that belongs to another pair or to the prime.
- The task-worktree condition is now `weftBranchExists(l, RecordsBranchName(branch))`. The records-worktree condition is now "`warp` exists locally".
- Case: the hub's prime sits on `develop`, and its records sibling is on `develop-weft`.
  - Slug A's task worktree is switched to the free branch `main`, and `main-weft` exists in the records repository.
  - The worktree step then returns `main` as A's branch, where before the change it returned `<prefix>A`.
  - Remove, discard and the PR lookup would target `main`.
- The records step has the mirror case: A's records worktree is on `main-weft` while `main` exists locally, so it returns `main`.
- The only remaining guards are the destructive gates (`ownedPairWarpBranch` and the landed-work check). I did not check whether those refuse `main`.
- The old row `TaskWorktreeOnAnUnrelatedBranchIsNotRead` passes only because `elsewhere` has no records branch. No row covers a worktree on a branch that does have one.
- This is the parent's decision as the fixer carried it out, so the finding is about how loose the recognition is. It is not about whether the decision should have been made.

**MEDIUM — `refuseWhenNothingRemains` (`internal/fabricengine/remove.go`) now asks origin for a launchers-only pair, including in `RemoveLocal`.**
- `refuseRemoval` is shared by `RemoveLocal`, whose doc says it "runs no ls-remote".
- Before the change, a pair with only its launcher directory left returned `"", nil` with no network call. Now `originPairBranch` runs, so `RemoveLocal` makes a network call.
- A failed lookup on a configured origin now fails `Remove` and `RemoveLocal` for that pair. Before, they would have proceeded.
- The code still adopts the origin name only when `!localMode`, but the origin ask happens in both modes.

**No findings** on F1 (`detachedBranch` is the same value the parser writes) or F4. The F4 test fails with the defect in place, because without the `rootErr == nil` guard `PairTaskBranch` would return the error.

**Tool calls:** none. My context held every file I needed.
