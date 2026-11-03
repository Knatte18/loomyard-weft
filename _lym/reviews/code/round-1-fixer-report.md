# Fixer-report, round 1

## Changes

- **F1** (fixed as proposed): `TestAddRollback_FinishesRefilledWeftWorktree` in `add_rollback_adopt_test.go` fails an Add with the launcher blocker, refills the weft directory after the rollback's clean git exit, and asserts the refill ran and the directory is gone.
- **F2** (fixed as proposed): row `a locked sibling worktree git refused to remove is returned and kept` in `TestRemove_FinishesWorktreeDirGitLeftOnDisk`.
  It asserts Remove fails, the sibling directory and its `.git` stay, and no `path_removed` is recorded.
- **F3** (fixed as proposed): row `a sibling worktree git deregistered whose removal fails is returned and kept`.
  It asserts the error wraps the removal error, names that git deregistered the directory, and the directory stays.
  The rows now share `wantErrContains` and a common failure branch.
- **F4** (fixed as proposed): `finishGitWorktreeRemoval` calls `List` itself and returns `gitErr` when listing fails or lists the target.
  The loop moved into a pure `listsLinkedWorktree`, which `isRegisteredLinkedWorktreeIn` still calls, so its other callers are unchanged.
  Not added: a listing-failure test, since no cheap fake exists for `List`.
- **F5** (fixed with own wording, from the self-check): the clean-exit error names the way forward, keeping `%w`.
  The reviewer's "then Remove run again" is qualified to "a pair that remains is then removed by running remove again", because the message is shared with Add's rollback, which leaves no pair.
- **F6** (fixed as proposed): `_lym/landing/summary.md` now says "Cause addressed", marks the cause as inferred and the refill as simulated, and notes the single check right after git exits.
  It also lists the new tests. This file is not committed, as `_lym` is not mine to commit.

Commits: one for F1, one for F2–F5 (they share `destroy.go` and one test file), one for the F5 rewording.
`lym gate test ./internal/fabricengine` and `--tier 1` pass.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal, LOW (F5 text assumes a Remove retry on Add's rollback path): fixed with own change, the F5 rewording above.
- fix-consistency, LOW (task-side nonzero-exit branch of `removeWarpWorktreeDir` still treats a failed listing as not registered): deferred, see below.
- fix-consistency, LOW (the task-side and deregistered-then-failed messages carry no way forward): deferred, see below.
- fix-consistency, LOW (wording tension between `isRegisteredLinkedWorktreeIn`'s doc and the new comment): no change, both are right for their callers.
- fix-affected-set, LOW (the rollback's warp-side `removeGitWorktree` and `prune.go` do not finish a leftover directory): deferred, see below.

## Disputed

None.

## Deferred

- Task-side nonzero-exit branch of `removeWarpWorktreeDir` with a failed listing, and its message: the review's Rejected list keeps that branch unchanged and the summary discloses the split, so changing it is the reviewer's and the orchestrator's call.
- Warp-side rollback removal and `prune.go`: no finding of the review names them, and the summary claims the finish only for the weft side.
