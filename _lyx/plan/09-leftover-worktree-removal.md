# Card 9 — leftover-worktree-removal

**Create:**
- `plan:internal/fabricengine#leftoverSeam` -> `type leftoverSeam struct`
- `plan:internal/fabricengine#removeLeftoverWorktreeDir` -> `func removeLeftoverWorktreeDir(rec *Mutations, container, target string, seam leftoverSeam) error`
- `plan:internal/fabricengine#leftoverRetryDelay` -> `const leftoverRetryDelay = 500 * time.Millisecond`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#SetLeftoverSeamForTest` -> `func SetLeftoverSeamForTest(t *Topology, remove func(container, target string, recursive bool) (removed, wasDir bool, err error), sleep func(time.Duration))`
- `plan:internal/fabricengine#RemoveContainedPathForTest` -> `var RemoveContainedPathForTest = removeContainedPath`
- `plan:internal/fabricengine#SampleLateWriterErrForTest` -> `var SampleLateWriterErrForTest error`
- `internal/fabricengine/latewriter_unix_test.go`
- `internal/fabricengine/latewriter_windows_test.go`
- `plan:internal/fabricengine#TestRemoveLeftoverWorktreeDir` -> `func TestRemoveLeftoverWorktreeDir(t *testing.T)`
- `plan:internal/fabricengine_test#TestRemove_FinishesADirectoryGitLeftOnDisk` -> `func TestRemove_FinishesADirectoryGitLeftOnDisk(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverSeam) error`
- `internal/fabricengine/export_test.go`
- `internal/fabricengine/destroy_test.go`
- `internal/fabricengine/remove_finish_integration_test.go`
- `internal/fabricengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D7: the task worktree's removal finishes a directory `git worktree remove` left refilled by a late writer.

`removeWarpWorktreeDir`, in its branch after a `*gitexec.GitError` where the target is no longer a registered linked worktree, now checks whether the target is still on disk (`pathPresent`).
When it is, it removes the target through `removeLeftoverWorktreeDir(rec, l.HubPath, target, leftover)`; success runs the best-effort `git worktree prune` and returns nil.
A failure returns today's not-a-linked-worktree error (git's exit code and stderr) with `; removing the directory it left failed: <err>` appended, the removal error wrapped with `%w`.
A target no longer on disk keeps today's error unchanged, a target git leaves still registered keeps today's gated `removePath` fallback, and the gate-refused-before-git branch above is untouched, so a target the gate refused is still refused (`TestRemove_RefusesForeignWorktreeWithoutDeletingIt` keeps pinning it).
Ownership proof for the leftover, replacing the registered-linked-worktree check: git ran, so the gate's ownership and dirtiness checks passed for this target when `Remove` began, and the target is the path git was asked to remove.
Recognition reads no stderr or message text: git exited nonzero, the target is unregistered and on disk, and the removal fails with a late-writer error value.

`removeLeftoverWorktreeDir` lives in `destroy.go` (`PATTERN-fabric-destruction-chokepoint`).
It removes `target` with `seam.remove` (nil: `removeContainedPath`, recursive) rooted at `container`, at most three attempts in total; between attempts it calls `seam.sleep` (nil: `time.Sleep`) with a fixed short delay (a `leftoverRetryDelay` constant of 500ms), and only when the failure satisfies `isLateWriterError`.
Any other failure, and a late-writer failure on the last attempt, is returned at once.
A successful removal of an entry appends `KindPathRemoved` for the target with detail `recursive` to `rec`, the entry `removePath` records for a directory (`PATTERN-mutation-record`); an already-absent target records nothing.
`leftoverSeam` holds `remove` and `sleep`; `Topology` gains a `leftover leftoverSeam` field, zero in production, which `Topology.Remove` passes to `removeWarpWorktreeDir`, its only caller.

`isLateWriterError` is declared once per OS: `latewriter_unix.go` (`//go:build !windows`) matches `syscall.ENOTEMPTY` with `errors.Is`; `latewriter_windows.go` matches `windows.ERROR_DIR_NOT_EMPTY` (a refilled directory) and `windows.ERROR_SHARING_VIOLATION` (a file under the target still open by its writer, such as a reed lock file `internal/lock` holds) with `errors.Is`, importing `golang.org/x/sys/windows`, already a module dependency.
Bound: only a late-writer failure is retried, at most twice more, only on the target git was asked to remove after the gate passed it, with an injected sleep.

Test support: `latewriter_unix_test.go` and `latewriter_windows_test.go` (package `fabricengine`, same build constraints) each set `SampleLateWriterErrForTest`, a `*os.PathError` wrapping the OS's not-empty value.
`export_test.go` keeps `RemoveWarpWorktreeDirForTest`'s four-argument shape, which `destructivegaps_integration_test.go` calls at two sites: the alias becomes a wrapper func that calls `removeWarpWorktreeDir` with a zero `leftoverSeam`, so those callers compile unchanged and its doc comment says the wrapper passes the production seam.
`export_test.go` also adds `SetLeftoverSeamForTest`, which sets `t.leftover` like `SetPushSeamForTest` sets the push seam (a nil argument keeps production for that part), and `RemoveContainedPathForTest = removeContainedPath`, so a fake remove can delegate to the real one.

Tests:
- `TestRemoveLeftoverWorktreeDir` in `destroy_test.go`, untagged, parallel, over a temp container with a target directory holding a file, counting sleeps:
  a late-writer failure on the first attempt then the real removal succeeds, with one sleep, the target gone and a `KindPathRemoved` `recursive` entry for it in `rec`;
  a late-writer failure on every attempt returns the error after three attempts and two sleeps, the record empty;
  any other failure (`os.ErrPermission`) is returned after one attempt with no sleep;
  on Windows only (skipped elsewhere, where an open file never blocks the removal), a file under the target held open through the real removal, with a sleep that closes it: the first attempt fails with the sharing violation, one sleep runs, and the target ends removed with the entry recorded.
- `TestRemove_FinishesADirectoryGitLeftOnDisk` in `remove_finish_integration_test.go` (`fabricengine_test`, integration tag), skipped on Windows and as root: a hubforge pair whose task worktree holds a committed file in a subdirectory that is then made read-only (0o555), so `git worktree remove` fails deleting it after git has deleted the registration, and `Remove` runs with force.
  Row `a late writer's refill is removed on a retry`: the leftover seam's remove fails once with `SampleLateWriterErrForTest`, then restores the subdirectory's mode and delegates to `RemoveContainedPathForTest`; `Remove` succeeds, the target is gone, the result's mutation record holds `KindPathRemoved` for the target with detail `recursive`, and one sleep ran.
  Row `any other removal failure is returned`: production removal (nil seam remove) meets the read-only subdirectory's permission error, `Remove` returns the composed error: it names the target as not a linked worktree, holds `removing the directory it left failed` and wraps the permission failure (`errors.Is(err, os.ErrPermission)`), and the target stays on disk; cleanup restores the mode.
- Two new top-level tests raise `internal/fabricengine`'s row in `cmd/lyx/testdata/test-budget.yaml` from 645 to 647.

Docs: `remove.go`'s file comment and `removeWarpWorktreeDir`'s doc comment, and `internal/fabricengine/doc.go`'s removal text, state the leftover branch, its ownership proof, the late-writer values per OS and the retry bound.

**ImpactSummary:** One new executor in `destroy.go`, reached only from `removeWarpWorktreeDir`'s not-registered branch, whose re-sign changes its single caller `Topology.Remove`; every other removal path and refusal is unchanged.

**Commit:** 9: Finish a task worktree directory git left behind after a late write
