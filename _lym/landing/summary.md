# fabric remove deletes worktree dirs refilled with ignored files

`lym fabric remove` now deletes a pair's worktree directory when it is still on disk after `git worktree remove` exited cleanly, on both the warp and the weft side.
A build or language server can write git-ignored output (`obj/`, `bin/`, caches) into the directory while git deletes it, or right after.
Before, `Remove` reported success and left a directory that was no longer a git repository, which made every batten verb on the run fail with "not a git repository".

Part of #21: this covers its first fault; the batten half is task `batten-stray-dir-and-beam-mark`.

## Design

- `finishGitWorktreeRemoval` in `internal/fabricengine/destroy.go` wraps `removeGitWorktree` at every site that tears down a pair worktree.
  Those sites are the warp worktree, `removeWeftWorktree`, `removeLocalWeft` and the `Add` rollback.
- It finishes the directory through the existing contained `removeWorktreeLeftover` in two cases:
  - git exited cleanly and the directory is back on disk;
  - git failed with a `GitError`, the worktree is no longer in `git worktree list`, and the directory remains.
- Any other outcome returns git's error unchanged.
  That covers a failure before git ran, a worktree still registered (a locked one, say), and a repository whose worktrees cannot be listed.
  A directory git still owns is never deleted recursively.
- If the finish itself fails, the error says the directory is no longer a worktree and should be deleted by hand; for a pair that remains, running remove again then removes it.
  The directory and its mutation record are left as they were.
- The `Add` rollback passes `t.leftover` to `removeWeftWorktree`, so a failed `Add` also removes a refilled weft directory.
- `leftoverSeam` gains a `refill` hook, and `SetLeftoverRefillForTest` sets it, so tests can play the late writer.
  `SetLeftoverSeamForTest` now sets only `remove` and `sleep`, keeping a refill set earlier.

## To check by hand

- Not run against a real C# hub: the new tests simulate the late writer.
  Run `lym fabric remove` on a pair whose worktree holds a populated `obj/` under a live build, and confirm both directories are gone.
- Tests added: `TestRemove_FinishesWorktreeDirGitLeftOnDisk` in `remove_leftoverdir_integration_test.go` and `TestAddRollback_FinishesRefilledWeftWorktree`.
  I did not run them in this session; run them through `lym gate test ./internal/fabricengine`.
