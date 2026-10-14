# Card 9 — leftover-worktree-retry

**Create:**
- `internal/fabricengine/latewriter_other.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#leftoverRemoval` -> `type leftoverRemoval struct`
- `plan:internal/fabricengine#leftoverRemoval.removeLeftover` -> `func (s leftoverRemoval) removeLeftover(rec *Mutations, container, target string) error`
- `plan:internal/fabricengine#RemoveWarpWorktreeDirWithLeftoverForTest` -> `func RemoveWarpWorktreeDirWithLeftoverForTest(rec *Mutations, l *lyxcwd.Location, target string, force bool, remove func(container, target string, recursive bool) (bool, bool, error), sleep func(time.Duration)) error`
- `plan:internal/fabricengine#RemoveContainedPathForTest` -> `var RemoveContainedPathForTest = removeContainedPath`
- `internal/fabricengine/remove_leftover_integration_test.go`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory` -> `func TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory(t *testing.T)`
- `internal/fabricengine/remove_leftover_windows_integration_test.go`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_RetriesASharingViolation` -> `func TestRemoveWarpWorktreeDir_RetriesASharingViolation(t *testing.T)`

**Edit:**
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverRemoval) error`
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/prune.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Finish a task worktree directory `git worktree remove` left behind after deleting its registration, retrying a late writer's refill (D7).

In `removeWarpWorktreeDir`, the branch after a `*gitexec.GitError` that finds the target no longer a registered linked worktree changes: when the target is still on disk (`pathPresent`), it calls `leftover.removeLeftover(rec, l.HubPath, target)` instead of returning at once.
A nil result returns nil, best-effort pruning as the registered fallback does; an error returns today's not-a-linked-worktree message with the removal's error added (`: %w`).
A target git leaves still registered keeps today's gated fallback, a target the gate refused before git ran is still refused by the earlier branch, and a target already gone returns today's message.
Ownership proof for the leftover, which replaces the registered-worktree check: git ran, so the gate's registered-linked-worktree and dirtiness checks passed for this target when this call began, and the target is the path git was asked to remove.

`leftoverRemoval` lives in `destroy.go`, the one file allowed a destructive primitive (`PATTERN-fabric-destruction-chokepoint`).
Its fields are `remove func(container, target string, recursive bool) (bool, bool, error)` and `sleep func(time.Duration)`; nil means `removeContainedPath` and `time.Sleep`, so the zero value is production.
`removeLeftover` removes the target recursively through `remove` under the hub container; when that fails with `isLateWriterError`, it sleeps `leftoverRetryDelay` (a new constant, 500ms) and tries the same target again, at most two more times.
On success it appends `KindPathRemoved` for the target with detail `recursive` (or `single` for a non-directory, as `removePath` does) to `Remove`'s record (`PATTERN-mutation-record`); an absent target records nothing.
Any other error, and a late-writer error after the last attempt, is returned.
`isLateWriterError` is per-OS and reads error values only, never git's stderr or any message text: `latewriter_other.go` (`//go:build !windows`) answers `errors.Is(err, syscall.ENOTEMPTY)`, and `latewriter_windows.go` answers `errors.Is` against `windows.ERROR_DIR_NOT_EMPTY` or `windows.ERROR_SHARING_VIOLATION` from `golang.org/x/sys/windows`.
Bound: only a late-writer failure is retried, only on the target git was asked to remove after the gate passed it, two retries with an injected sleep; nothing else is deleted or retried.

`Topology` gains a `leftover leftoverRemoval` field (zero in production), which `Topology.Remove` passes on.
`RemoveWarpWorktreeDirForTest` becomes a function with today's four parameters that passes a zero `leftoverRemoval`, so its existing callers are unchanged; `RemoveWarpWorktreeDirWithLeftoverForTest` passes the given `remove` and `sleep`, and `RemoveContainedPathForTest` exposes the real removal for a seam that delegates to it.

Comments the change makes wrong, reworded in this card:
- `remove.go`'s file comment, whose last sentence says Remove never deletes a directory git declined to remove unless it is a registered linked worktree: it adds that a directory git deleted the registration of, and left on disk after failing its own delete, is finished through the leftover removal (the file comment sits outside `removeWarpWorktreeDir`, whose glyph this card lists).
- `doc.go`'s sentence naming "the two fallbacks that fire *after* git has already declined the removal" as the only recursive removals with a real dirtiness scope: it names the leftover removal as a third recursive removal after a git failure, whose dirtiness proof is the gate run before git.
- `prune.go`'s file comment, which calls its rule "the same rule removeWarpWorktreeDir applies": it says the warp side also finishes a leftover git unregistered, which prune has no counterpart for.

Tests: the two new files are `package fabricengine_test`, like their siblings, and drive the `…ForTest` re-exports.
`TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory` (`integration` tag, skipped on Windows and as root) builds the state git leaves: a `hubforge` pair with an untracked `stuck/` subdirectory holding a file, the subdirectory made `0o555`, removed with `force` true, so `git worktree remove --force` fails deleting the file, deletes the registration and exits nonzero, leaving the target on disk.
Each row restores the mode in cleanup and counts the seam's calls and sleeps:
- `a refill that clears on a retry is removed`: the seam returns a `*fs.PathError` wrapping `syscall.ENOTEMPTY` once, then restores `0o755` and calls `RemoveContainedPathForTest`; the call returns nil, the target is gone, one sleep ran, and the record's `Entries()` hold `KindPathRemoved` for the target with detail `recursive`.
- `a persistent not-empty failure is returned after the last attempt`: the seam always returns `ENOTEMPTY`; three calls, two sleeps, an error naming the target and wrapping `syscall.ENOTEMPTY`, the target still on disk.
- `another removal failure is not retried`: the seam delegates to `RemoveContainedPathForTest` without restoring the mode; one call, no sleep, an error.
The criterion that a target the gate refused before git ran is still refused stays with the existing `TestRemove_RefusesForeignWorktreeWithoutDeletingIt`, which drives that unchanged branch through `Topology.Remove` and asserts the refusal and the surviving content.
`TestRemoveWarpWorktreeDir_RetriesASharingViolation` (`integration && windows`) builds a pair, holds a file under the target open with `os.Open` (no share-delete), and removes with `force`: git fails on the open file and drops the registration, the first leftover attempt fails with a sharing violation, the test's `sleep` closes the file, and the retry removes the target.
It runs only on a Windows host; the card's `Verify:` cross-compiles it and `latewriter_windows.go` from this host, so a wrong `windows.ERROR_*` spelling fails here.
`test-budget.yaml`'s `internal/fabricengine` row rises by two.
`doc.go` adds the leftover removal, its ownership proof and its bound beside the fallback text.

**ImpactSummary:** Re-signs one private remover whose callers (Topology.Remove and its test re-export) change on this card; adds one Topology seam field; only the git-failed-and-unregistered branch gains behavior.

**Verify:** GOOS=windows CGO_ENABLED=0 go vet -tags integration ./internal/fabricengine
