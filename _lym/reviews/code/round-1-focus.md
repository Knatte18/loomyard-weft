---
round: 1
focus:
  - "internal/fabricengine/destroy.go `finishGitWorktreeRemoval`: does each branch return the right error? Check the nonzero-git paths (non-GitError, still registered, nothing on disk) against what each caller did before, and whether the wrapped message still lets callers' `errors.As` on a refusal or GitError type work."
  - "Containment: every caller passes `l.HubPath` as container for both the task worktree and the weft sibling path. Settle whether each target (`RecordsWorktreePath`, the task worktree) always lies under `l.HubPath`, so `removeContainedPath`'s os.Root does not refuse a legitimate sibling."
  - "Sibling-side behavior change in `removeWeftWorktree`, `removeLocalWeft` and Add's `rollbackAdd`: a deregistered-but-present weft directory is now deleted instead of failing Remove. Settle whether that stays inside the ownership proof for all three callers, including rollback of a reused or adopted weft, and whether the diff's doc comments match."
  - "`removeLocalWeft` and `removeSibling`: after finishGitWorktreeRemoval returns an error, the `pathPresent` checks still decide failure. Check that a nil return with the directory gone cannot leave a stale error path, and that the step lists reported to the operator include the finished removal."
  - "`removeWarpWorktreeDir`: the clean-exit path now calls finishGitWorktreeRemoval, while the existing nonzero-exit handling is left alone. Settle whether the two paths can disagree on ownership, and whether the old nonzero path should have been routed through the same helper."
  - "The `refill` field on production `leftoverSeam` and `SetLeftoverSeamForTest` now assigning fields instead of replacing the struct: settle that a nil refill is a no-op in production and that existing seam tests still set what they relied on."
  - "Scope against the Goal: the board entry asks that `fabric remove` delete the whole worktree directory, ignored files included, for the task and weft sibling. Settle whether the diff does that for every removal path (Remove, local-only pair, Add rollback) and licenses nothing further."
  - "Test economy in `remove_leftoverdir_integration_test.go`: for each row, name the existing test in the package that already covers the same leftover behavior, if any, and whether each row reaches a distinct code path. Check the read-only-directory row's platform guard and cleanup."
  - "Summary claims in `_lym/landing/summary.md` (sibling now finishes a git-failed directory; Add rollback gets the same finish; rows asserting `path_removed` and the matching step): check each against the diff and the test."
  - "Repository rules: comment conventions on the new and changed doc comments in destroy.go, remove.go, weftwiring.go, export_test.go, and whether the package doc.go paragraph change is what the repo's docs rules require for this change."
---

Seed focus for round 1. The decision record carries only the board entry's `## Goal`, so the answer key is that entry alone.
