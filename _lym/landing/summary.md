# fabricengine: Add deletes a task path holding only git-ignored output

`Topology.Add` no longer halts when the task worktree path already exists as a directory of build output.
A build or IDE can recreate `obj/` directories under the path of a removed pair.
A later pair with the same slug, such as a repeated `lym beam merge-parent`, then failed with `*ErrTaskPathTaken`
until the operator deleted the directory by hand.

Closes #108

## What changes

- `Add` deletes the directory at the task worktree path and makes the pair there when the directory is provably build output.
  The proof is `git check-ignore`: the directory is no registered worktree, holds no `.git` entry at any depth,
  and every file under it is untracked and ignored at the same relative path in the acting worktree.
  An empty directory counts.
- Any other path still returns `*ErrTaskPathTaken`: a registered worktree, a non-directory,
  a directory with a `.git` entry or with one file git does not ignore.
  The refusal text now says fabric deletes only a directory of git-ignored output.
- A step of the judgement that fails (listing worktrees, walking the directory, `git check-ignore`) keeps the path,
  logs a warning and returns `*ErrTaskPathTaken`.
- A delete that fails returns an error wrapping `ErrStrayPathKept` and leaves the path on disk.
  `ErrStrayPathKept` was previously returned only after a pair's removal, and its doc comment says so.
- The delete is recorded as a `KindPathRemoved` mutation and logged at info level.

## Design decisions

- The fix is in `Add`, not in teardown.
  A build can write into the path at any time after teardown, so only the create side can be sure to meet the leftover.
- The judgement runs in the pre-flight, but the delete runs after the remaining pre-flight refusals have passed,
  so an `Add` that is refused for another reason leaves the directory untouched.
- The path is judged again just before the delete, since the pre-flight can take a network round;
  a file saved there in between keeps the directory.
- The delete reuses `removeWorktreeLeftover`, so it keeps `os.Root` containment and the retry against a late writer.
  It is a third documented exception, beside `RemoveStrayTaskPath`, to the executor order in `PATTERN-fabric-boundary`:
  ownership comes from `git check-ignore`, and `--force` and dirtiness do not apply.
  `PATTERN-fabric-boundary.md` and the `doc.go` of `fabricengine` state it.

## To check by hand

Nothing was run by hand.
The new tests are in `add_taskpathleftover_integration_test.go`.
Check these two findings from the code review:

- `Add` deleting before the pre-flight refusals (`internal/fabricengine/add.go`):
  confirm the delete now sits after the last refusal and that `TestAdd_TaskPathLeftoverKeptWhenPreflightRefuses` pins it.
- The refusal for a tracked file under the path (`internal/fabricengine/destroy.go`, `ignoresEveryPath`):
  confirm the `TrackedFileKeepsIt` case in `TestAdd_TaskPathLeftover` pins it.
