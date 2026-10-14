# Card 8 — Task worktree removal finishes a directory `git worktree remove` left refilled

**Create:**
- `plan:internal/fabricengine#leftoverSeam` -> `type leftoverSeam struct`
- `plan:internal/fabricengine#removeLinkedWorktree` -> `func removeLinkedWorktree(rec *Mutations, req pathRequest, repoDir string, leftover leftoverSeam) (leftoverErr, err error)`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#RemoveWarpWorktreeDirWithLeftoverSeamForTest` -> `func RemoveWarpWorktreeDirWithLeftoverSeamForTest(rec *Mutations, l *lyxcwd.Location, target string, force bool, sleep func(time.Duration), remove func(container, target string) (removed, wasDir bool, err error)) error`
- `internal/fabricengine/remove_leftover_integration_test.go`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory` -> `func TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory(t *testing.T)`

**Edit:**
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverSeam) error`
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `internal/fabricengine/doc.go`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D7.
When git cannot fully delete a linked worktree's directory, `git worktree remove` exits nonzero after already deleting the worktree's registration, and leaves the directory on disk.
`removeWarpWorktreeDir` then finds the target unregistered and refuses, so today's fallback is never reached and a refilled task worktree directory stays behind.

New executor `removeLinkedWorktree`, in `destroy.go` (`PATTERN-fabric-destruction-chokepoint`):
- It calls `removeGitWorktree(rec, req, repoDir)`, which runs the gate's pipeline for `req` and then git.
- On success, or on an error that is not a `*gitexec.GitError` (the gate refused or git never ran), it returns `(nil, err)` unchanged.
- On a `GitError` it checks the leftover condition: the target is no longer a registered linked worktree of `repoDir` (`isRegisteredLinkedWorktreeIn`) and is still on disk (`os.Lstat`).
  Only then does it remove `req.target` recursively through the contained removal rooted at `req.container`, which is `removeContainedPath` unless the seam replaces it.
  - When that removal fails with a late-writer error (`isLateWriterError`), it sleeps and removes the same target again, at most two more times, sleeping one second and then two.
  - On success it appends `KindPathRemoved` for the target with detail `recursive` (`single` for a non-directory), the entry the gated `removePath` records, and returns `(nil, nil)`.
  - A failure after the last attempt, or any other removal failure, returns at once as `(removeErr, gitErr)`.
- A target git leaves registered, or one already gone, returns `(nil, gitErr)`, keeping today's fallback and messages.

Ownership proof (the record's): the leftover removal runs inside the same executor whose pipeline passed `req`'s registered-linked-worktree, containment and dirtiness checks just before git ran on `req.target`.
That replaces the registered-linked-worktree check for this one directory, and git had already accepted discarding the work tree's content.
Neither half of the recognition reads git's stderr or any message text.
`isLateWriterError` is per-OS:
- `latewriter_unix.go` (`//go:build !windows`): `errors.Is(err, syscall.ENOTEMPTY)`.
- `latewriter_windows.go`: `errors.Is` against `windows.ERROR_DIR_NOT_EMPTY` or `windows.ERROR_SHARING_VIOLATION` from `golang.org/x/sys/windows`.
  The second covers a file under the target still open by its writer, such as a reed lock file held by `internal/lock`; Go opens Windows files without `FILE_SHARE_DELETE`.
Bound: only a late-writer failure is retried, only on the target git was asked to remove after the gate passed it, at most three attempts, with an injected sleep.

`leftoverSeam` holds `sleep func(time.Duration)` and `remove func(container, target string) (removed, wasDir bool, err error)`.
A nil field means production: `time.Sleep` and `removeContainedPath(container, target, true)`.
`Topology` gains a `leftover leftoverSeam` field, zero in production, and `Topology.Remove` passes `t.leftover` to `removeWarpWorktreeDir`.
`removeWarpWorktreeDir` calls `removeLinkedWorktree` in place of `removeGitWorktree`.
On success it returns nil.
In its not-a-linked-worktree branch it keeps today's message and, when `leftoverErr` is non-nil, adds `"; removing the directory git left failed: <leftoverErr>"`.
Every other branch is unchanged, and a target the gate refused before git ran is still refused with today's message.
A re-entered `Remove` no longer meets a stray leftover from this cause, and `Remove`'s steps report the task worktree removed once the target is gone.

Test seams in `export_test.go`:
- `RemoveWarpWorktreeDirForTest` becomes a four-argument function value calling `removeWarpWorktreeDir` with a zero `leftoverSeam`, so its two existing callers in `destructivegaps_integration_test.go` stay unchanged.
- The new `RemoveWarpWorktreeDirWithLeftoverSeamForTest` passes the two seams through.

Docs and guards:
- `Topology.Remove`'s doc comment adds that a task worktree directory git leaves unregistered during this `Remove` is removed, unlike a stray path found before it.
- `fabricengine/doc.go` adds `removeLinkedWorktree` to the executor list under "Why the gate executes rather than approves".
  Its dirtiness-window paragraph's "the two fallbacks that fire *after* git has already declined the removal" gains the leftover removal as a third site.
  It gains a paragraph stating the leftover ownership proof, the late-writer values per OS and the bound.
- `cmd/lyx`'s `destructiveGuardRecordingExecutors` gains `{"removeLinkedWorktree", "func removeLinkedWorktree(rec *Mutations, "}` (`PATTERN-mutation-record`).
- `cmd/lyx/testdata/test-budget.yaml`: `internal/fabricengine` rises by one for the new test.

Test `TestRemoveWarpWorktreeDir_FinishesALeftoverDirectory` (`//go:build integration`, package `fabricengine_test`, parallel rows).
Each row starts from `hubforge.CopyHub` and `hubforge.AddPairWith(..., AddOptions{SkipPush: true})`, as the neighbouring fallback tests do.
It records `fabricengine.NewMutations("")` and counts sleep calls.
The unix rows run with `force` true and skip on Windows and as root.
They make git fail partway by creating `<target>/ro/held.txt` and setting `<target>/ro` to `0o555`, with a cleanup restoring `0o755`.
Git's `worktree remove --force` then exits nonzero, having deleted the registration, with the directory on disk.
Rows:
- "a late writer once, then the leftover is removed": the remove seam's first call restores `ro` to `0o755` and returns an `*os.PathError` wrapping `syscall.ENOTEMPTY`.
  Its later calls `os.RemoveAll(target)` and report removed.
  It wants nil, the target gone, one sleep, and one `KindPathRemoved` entry for the target with detail `recursive`.
- "a late writer past the last attempt is returned": the seam always returns `ENOTEMPTY`.
  It wants an error naming "not a linked worktree of this repo" and "removing the directory git left failed", three remove calls, two sleeps, the target on disk and no `KindPathRemoved`.
- "another removal failure is not retried": production removal, which hits `EACCES` under `ro`.
  It wants an error naming the removal failure, no sleep, and the target on disk.
- "a target the gate refused before git ran is still refused": a plain directory under the hub that is not a worktree, run on every OS.
  It wants today's "it is not a linked worktree of this repo" refusal, the directory on disk, and no sleep and no remove call.
- "a file held open by its writer is removed once the writer closes it", Windows only:
  `<target>/held.lock` is created and kept open, and production removal is used.
  The sleep seam closes the file on its first call.
  It wants nil, the target gone, one sleep and the `KindPathRemoved` entry.
  If Go's root-relative delete surfaces the held file as `ERROR_ACCESS_DENIED` rather than a sharing violation, the row fails, and the implementer reports the observed error instead of widening `isLateWriterError` past the record's two values.

**ImpactSummary:** The task worktree removal gains one executor path that removes a directory git left unregistered after a failed `git worktree remove`, retried only on a late-writer error; every other removal branch, the gate's refusals and the other `removeGitWorktree` callers are unchanged.

**Commit:** 8: fabric: finish a task worktree directory git left behind, retrying a late writer
