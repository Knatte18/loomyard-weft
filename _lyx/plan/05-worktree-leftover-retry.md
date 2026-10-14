# Card 5 — worktree-leftover-retry

**Create:**
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#leftoverRemoval` -> `type leftoverRemoval struct`
- `plan:internal/fabricengine#leftoverRetryWait` -> `const leftoverRetryWait = time.Second`
- `plan:internal/fabricengine#pathOwnershipGitRemoveLeftover` -> `const pathOwnershipGitRemoveLeftover pathOwnershipKind = iota`
- `plan:internal/fabricengine#ownedGitRemoveLeftover` -> `func ownedGitRemoveLeftover(gitErr *gitexec.GitError, repoDir string) pathOwnership`
- `plan:internal/fabricengine#removeWorktreeLeftover` -> `func removeWorktreeLeftover(rec *Mutations, req pathRequest, leftover leftoverRemoval) error`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_FinishesTheDirectoryGitLeft` -> `func TestRemoveWarpWorktreeDir_FinishesTheDirectoryGitLeft(t *testing.T)`
- `plan:internal/fabricengine#RemoveWarpWorktreeDirWithLeftoverForTest` -> `func RemoveWarpWorktreeDirWithLeftoverForTest(rec *Mutations, l *lyxcwd.Location, target string, force bool, sleep func(time.Duration), remove func(container, target string, recursive bool) (bool, bool, error)) error`

**Edit:**
- `internal/fabricengine#pathOwnership`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverRemoval) error`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D7: when `git worktree remove` exits nonzero having already dropped the task worktree's registration and left its directory on disk, `removeWarpWorktreeDir` finishes the directory, retrying a late-writer failure, instead of returning its not-a-linked-worktree error.

`removeWarpWorktreeDir` (`internal/fabricengine/remove.go`) gains a `leftover leftoverRemoval` parameter.
In its branch after a `*gitexec.GitError` where `isRegisteredLinkedWorktree(l, target)` is false, when `pathPresent(target)` holds it calls `removeWorktreeLeftover` with a `pathRequest` of what `"remove warp worktree leftover"`, container `l.HubPath`, target `target`, ownership `ownedGitRemoveLeftover(gitErr, l.WorktreePath())` and dirtiness `dirtinessNA("git worktree remove already accepted discarding the work tree's content")`.
Success runs the best-effort `git worktree prune` the existing fallback runs and returns nil.
Failure returns today's not-a-linked-worktree error, its git exit and stderr, with the leftover removal's error added (`%w`).
A target git left still registered keeps today's `removePath` fallback, a target git left absent returns today's error, and a target the gate refused before git ran is still refused in the branch above, unchanged.
`Topology.Remove`, the one production caller, passes `leftoverRemoval{}`, the production zero value.

In `destroy.go`, the one file permitted a destructive primitive (`PATTERN-fabric-destruction-chokepoint`):
- `pathOwnershipGitRemoveLeftover` is appended as the last member of the existing `pathOwnershipKind` iota block, and `pathOwnership` gains a `gitErr *gitexec.GitError` field that `ownedGitRemoveLeftover` alone sets.
- `resolvePathOwnership`'s new case answers owned only when `gitErr` is non-nil, its `Dir` is `repoDir`, its `Args` are `worktree remove`, optionally `--force`, then exactly the target, and the target is not a registered linked worktree of `repoDir`.
  This is the decision's ownership proof: git ran on exactly this target after the gate's registered-linked-worktree and dirtiness checks passed it, and git has already given the target up.
  The predicate reads no stderr or message text.
  Its mismatch branches (a nil `gitErr`, another `Dir`, `Args` naming another path) are a fail-loud wiring guard in `refusal-spec.md`'s sense: `removeWarpWorktreeDir` builds the request from the `GitError` its own `removeGitWorktree` call on `target` returned, so no input reaches them and they take no per-row test.
- `leftoverRemoval` holds `sleep func(time.Duration)` and `remove func(container, target string, recursive bool) (removed, wasDir bool, err error)`; nil fields mean `time.Sleep` and `removeContainedPath`.
- `removeWorktreeLeftover` runs `checkPathRequest` once (containment, ownership, dirtiness, in order), then removes the target recursively through the contained removal at most three times: when an attempt's error satisfies `isLateWriterError` and attempts remain, it sleeps `leftoverRetryWait` and tries again.
  On success with an entry removed it appends `KindPathRemoved` for the target with detail `recursive` for a directory and `single` otherwise, as `removePath` records, so `Remove`'s mutation record shows the deleted directory (`PATTERN-mutation-record`).
  The last late-writer error, or any other removal error on its first occurrence, is returned wrapped; a gate refusal is returned unwrapped as every executor's is.
Bound: only a late-writer failure is retried, at most twice, only on the target git was asked to remove after the gate passed it, with the sleep injectable.

`isLateWriterError` is per OS, matched with `errors.Is`: `latewriter_unix.go` (`//go:build !windows`) answers `syscall.ENOTEMPTY`; `latewriter_windows.go` answers `windows.ERROR_DIR_NOT_EMPTY` and `windows.ERROR_SHARING_VIOLATION` from `golang.org/x/sys/windows`, already a module requirement.
Neither file holds a destructive primitive.

Tests:
Every row spawns git, since the ownership case asks `isRegisteredLinkedWorktreeIn`, so all of them live in the `integration` tier, per `destroy_test.go`'s tier note; no untagged test is added.
- `TestRemoveWarpWorktreeDir_FinishesTheDirectoryGitLeft`, a new top-level table test in `destructivegaps_integration_test.go` (package `fabricengine_test`, build tag `integration`) on a `hubforge` pair, drives `RemoveWarpWorktreeDirWithLeftoverForTest` with a fresh `Mutations` recorder and a `sleep` hook that records each wait.
  On Linux and macOS (rows skipped on Windows and as root), git is made to fail by a tracked subdirectory of the task worktree set to `0o555`, so `git worktree remove --force` cannot delete a file in it, exits nonzero, drops the registration and leaves the directory; a `t.Cleanup` restores `0o755` on that subdirectory, so the temporary tree is always removable whatever the row's outcome.
  Rows:
  - the fake `remove` returns `syscall.ENOTEMPTY` on its first call and calls the real removal after, and the `sleep` hook restores `0o755`, standing in for the late writer finishing: nil error, one recorded wait of `leftoverRetryWait`, the target gone, and one `KindPathRemoved` entry with detail `recursive` for the target in the record;
  - the fake `remove` returns `syscall.ENOTEMPTY` on every call: an error after three calls and two recorded waits, naming the target as not a linked worktree with the removal's error added, the target still on disk and no `KindPathRemoved` entry;
  - no fake `remove`, the permission left in place: the real removal fails with a permission error, which is not retried (no recorded wait), and the error names the target as not a linked worktree with the removal's error added.
  - On Windows only (skipped elsewhere): git is made to fail by a file under the target that the test holds open, which Go opens without `FILE_SHARE_DELETE`; there is no fake `remove`, and the `sleep` hook closes the file, standing in for the writer closing it: the first real leftover attempt fails with a sharing violation, the second removes the target, with one recorded wait and one `KindPathRemoved` entry. This row runs only on a Windows host; on Linux the card's `Verify:` compiles the Windows files.
- In `export_test.go`, `RemoveWarpWorktreeDirForTest` becomes a function with its current parameters that passes `leftoverRemoval{}`, so the existing integration callers are unchanged; `RemoveWarpWorktreeDirWithLeftoverForTest` builds a `leftoverRemoval` from its two seams.
- The existing `TestRemove_RefusesForeignWorktreeWithoutDeletingIt` and `TestRemove_RefusesPrimeWorktreeAndLeavesItIntact` keep pinning that a target the gate refused before git ran is still refused.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/fabricengine` from 645 to 646.

`internal/fabricengine/doc.go`'s description of the fallbacks that fire after git declined a removal gains the leftover removal, its ownership proof and its bounded late-writer retry.

**ImpactSummary:** One new executor and ownership kind in `destroy.go`, one new branch in `removeWarpWorktreeDir` reached only after git exited nonzero and gave the target up; its one production caller `Topology.Remove` passes the zero seam; the test export keeps its signature; two per-OS predicate files; one budget row.

**Verify:** GOOS=windows go vet -tags integration ./internal/fabricengine
