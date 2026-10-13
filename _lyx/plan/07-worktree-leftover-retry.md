# Card 7 — Finish a task worktree directory `git worktree remove` left refilled

**Create:**
- `plan:internal/fabricengine#leftoverSeam` -> `type leftoverSeam struct`
- `plan:internal/fabricengine#removeWorktreeLeftover` -> `func removeWorktreeLeftover(rec *Mutations, container, target string, seam leftoverSeam) error`
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#SetLeftoverSeamForTest` -> `func SetLeftoverSeamForTest(t *Topology, remove func(container, target string) (removed, wasDir bool, err error), sleep func(delay time.Duration))`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_LeftoverRetry` -> `func TestRemoveWarpWorktreeDir_LeftoverRetry(t *testing.T)`

**Edit:**
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverSeam) error`
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `internal/fabricengine/doc.go`
- `pattern/PATTERN-fabric-destruction-chokepoint.md`
- `internal/fabricengine_test#TestRemove_RefusesForeignWorktreeWithoutDeletingIt`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#removeContainedPath`

**Intent:**
D7: when `git worktree remove` exits nonzero having already deleted the task worktree's registration, and the directory it could not fully delete is still on disk, `removeWarpWorktreeDir` finishes the removal instead of returning its not-a-linked-worktree error, retrying a late writer's refill a bounded number of times.

`removeWarpWorktreeDir`'s branch after a `*gitexec.GitError` with the target no longer a registered linked worktree splits on `pathPresent(target)`: a target still on disk goes to `removeWorktreeLeftover(rec, l.HubPath, target, leftover)`, and an absent one keeps today's error.
Ownership proof, replacing the registered-linked-worktree check for this leftover only: git ran, so the gate's registered-linked-worktree and dirtiness checks passed for this target when this `Remove` began, and the target is the path git was asked to remove, which git had already accepted discarding.
A target the gate refused before git ran keeps its refusal, since that branch returns before the git-error branch, and a target git leaves still registered keeps today's gated `removePath` fallback.
`removeWorktreeLeftover` lives in `destroy.go`, per `PATTERN-fabric-destruction-chokepoint`, and its doc comment states that the caller's git run is its ownership proof.
It removes the target recursively through `seam.remove`, nil meaning `removeContainedPath(container, target, true)`, which keeps containment under the hub through `os.Root`.
When that fails with an error `isLateWriterError` accepts, it waits through `seam.sleep`, nil meaning `time.Sleep`, one second and then two, and removes the same target again, at most three attempts in all.
A successful removal appends `KindPathRemoved` for the target with detail `recursive`, the entry `removePath` records for a directory, so `Remove`'s mutation record shows the deleted directory, per `PATTERN-mutation-record`.
`destructiveGuardRecordingExecutors` in `cmd/lyx/destructiveguard_test.go` gains the row `{"removeWorktreeLeftover", "func removeWorktreeLeftover(rec *Mutations, "}`, so `TestMutationRecord_FabricengineProductionSource` pins its leading recorder.
A failure after the last attempt, or any failure `isLateWriterError` rejects on its first attempt, is returned unretried as the not-a-linked-worktree error today's branch returns, with git's exit code and stderr and the removal's error wrapped with `%w`.
Bound: only a late-writer failure is retried, only on the target git was asked to remove after the gate passed it, at most twice, and no git stderr or message text is read.

`isLateWriterError` matches with `errors.Is`: `latewriter_unix.go` (`//go:build !windows`) accepts `syscall.ENOTEMPTY`, and `latewriter_windows.go` accepts `windows.ERROR_DIR_NOT_EMPTY` (a refilled directory) and `windows.ERROR_SHARING_VIOLATION` (a file under the target still held open by its writer, such as a reed lock file `internal/lock` holds, since Go opens Windows files without `FILE_SHARE_DELETE`), from `golang.org/x/sys/windows`, already a module dependency.

`Topology` gains `leftover leftoverSeam`, zero value production like its `push` seam, and `Topology.Remove` passes `t.leftover` to `removeWarpWorktreeDir`.
`leftoverSeam` holds `remove func(container, target string) (removed, wasDir bool, err error)` and `sleep func(delay time.Duration)`.
In `export_test.go`, `RemoveWarpWorktreeDirForTest` changes from a `var` alias to a function of the same four parameters that passes the zero seam, so its two existing callers, the `TestRemoveWarpWorktreeDir_Fallback…` tests, keep their call as written, and `SetLeftoverSeamForTest` sets a `Topology`'s seam per instance, as `SetPushSeamForTest` does.
`fabricengine/doc.go`'s `Remove` paragraph says a directory git deregistered but left on disk is finished through the contained removal, with the late-writer retry and its bound.
Its `destroy.go` paragraph, which says every destructive primitive runs the four-check pipeline first, and the background file `pattern/PATTERN-fabric-destruction-chokepoint.md`, which says every executor checks containment, ownership, dirtiness and force, both name the one exception: `removeWorktreeLeftover` keeps containment through `os.Root`, and takes ownership and dirtiness from the gated `git worktree remove` its caller just ran on the same target.
The `PATTERN.md` entry line is unchanged.

Tests: `TestRemoveWarpWorktreeDir_LeftoverRetry`, a new table test in `destructivegaps_integration_test.go` (`//go:build integration`, package `fabricengine_test`), drives `Topology.Remove` on a `hubforge` pair with `SetLeftoverSeamForTest`, and reads `Remove`'s result's mutation record.
On Linux and macOS, `git worktree remove` is made to fail by a read-only subdirectory under the target holding a file committed on the pair's branch, so the pair stays clean and passes `Remove`'s dirty checks (a directory's mode is not tracked): git deletes the registration, fails the unlink and leaves the directory.
These rows skip on Windows and when run as root, where a read-only directory does not stop the delete, and restore the subdirectory's mode in cleanup.
Rows:
- "a leftover whose first removal reports not-empty is removed on a retry": the fake `remove` answers `syscall.ENOTEMPTY` once, then restores the mode and deletes the target; `Remove` succeeds, the target is gone, the record holds `KindPathRemoved` with detail `recursive` for the target, and the fake sleep ran once.
- "a leftover still not empty after the last attempt is returned": the fake always answers `syscall.ENOTEMPTY`; the error wraps it (`errors.Is`) and names the target as not a linked worktree, `remove` ran three times, sleep twice, and the target is on disk.
- "another removal failure is not retried": the fake answers a permission error; the error wraps it, `remove` ran once, sleep never.
- On Windows only, "a file held open by its writer is removed once the writer closes it": a file committed on the pair's branch, so the pair stays clean, is held open with `os.Open`, so git's delete fails and the first contained removal fails with `ERROR_SHARING_VIOLATION`; the seam keeps the real removal and its fake sleep closes the file, so the retry succeeds, with the `KindPathRemoved` entry.
"A target the gate refused before git ran is still refused" is pinned by `TestRemove_RefusesForeignWorktreeWithoutDeletingIt`, which already drives `Topology.Remove` on a hub directory that is no warp linked worktree and asserts the refusal and the surviving content; it gains a counting seam set with `SetLeftoverSeamForTest` and asserts the seam's `remove` and `sleep` never ran.
`test-budget.yaml`'s `internal/fabricengine` row rises by one for the new test.

**ImpactSummary:** `removeWarpWorktreeDir` is re-signed with its one production caller, `Topology.Remove`, and its test export on this card; the new removal stays inside `destroy.go` and changes only the branch where git deregistered a target it left on disk.

**Commit:** 7: Finish a task worktree directory git deregistered but left refilled
