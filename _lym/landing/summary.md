# fabric remove deletes a worktree directory refilled after git removed it

From issue #21: on a C# hub, `lym fabric remove` succeeded but left the task worktree directory on disk holding `obj/` build output under three projects, and every batten verb then failed on that directory with "not a git repository".

## Cause

A clean `git worktree remove` already deletes ignored files.
What it leaves is a directory a late writer refills once git has deleted it: a C# language server or build server running against the worktree writes `obj/project.assets.json` and its siblings again.
Remove looked for a leftover directory only when git exited nonzero, so after a clean exit the refilled directory stayed and the removal still reported success.

## Change

- `finishGitWorktreeRemoval` (`internal/fabricengine/destroy.go`) completes a `git worktree remove` given the error git returned.
  After a clean exit with the target still on disk, or after a git failure that left the target deregistered and on disk, it removes the directory through `removeWorktreeLeftover`, which keeps containment through `os.Root` and retries a late writer's failure.
  Any other outcome returns git's error unchanged.
- The task worktree (`removeWarpWorktreeDir`) calls it after git's clean exit. Its handling of a nonzero exit is unchanged.
- The sibling worktree calls it after every git run, in `removeWeftWorktree` and in the local-only teardown `removeLocalWeft`.
  The sibling side therefore also finishes a directory git deregistered and failed to delete, which before this change failed Remove with "the weft worktree remains".
  Add's rollback goes through `removeWeftWorktree` too and gets the same finish.
- `leftoverSeam` gains a `refill` hook, nil in production, that a test uses to play the late writer after git's clean exit.
- The package doc's teardown paragraph names both kinds of leftover directory.

The ownership proof is the same as for the existing leftover path: the gated `git worktree remove` already passed the registered-linked-worktree and dirtiness checks on this target, and git dropped the registration.

## Tests

`TestRemove_FinishesWorktreeDirGitLeftOnDisk` (`remove_leftoverdir_integration_test.go`) drives `Topology.Remove` over a real pair. Its rows cover:

- a task worktree refilled after git's clean exit;
- a sibling worktree refilled after git's clean exit;
- a local-only pair's sibling refilled after git's clean exit;
- a sibling git deregistered and failed to delete, using a read-only directory, off Windows only;
- a refilled task worktree whose removal fails: Remove returns the error and the directory stays.

Each successful row asserts that the directory is gone, that a `path_removed` mutation is recorded, and that the matching step is reported.

## Out of scope

The batten half of the issue is task `batten-stray-dir-and-beam-mark`: batten treating a directory that is not a linked worktree as a gone pair, and the beam-mark refusal.
