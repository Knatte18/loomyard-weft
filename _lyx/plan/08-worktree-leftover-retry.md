# Card 8 — worktree-leftover-retry

**Create:**
- `plan:internal/fabricengine#leftoverSeam` -> `type leftoverSeam struct`
- `plan:internal/fabricengine#leftoverRetries` -> `const leftoverRetries = 2`
- `plan:internal/fabricengine#leftoverRetryDelay` -> `const leftoverRetryDelay = 2 * time.Second`
- `plan:internal/fabricengine#removeWorktreeLeftover` -> `func removeWorktreeLeftover(rec *Mutations, req pathRequest, gitErr *gitexec.GitError, seam leftoverSeam) error`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#lateWriterErrors` -> `var lateWriterErrors []error`
- `plan:internal/fabricengine#TestGate_WorktreeLeftover` -> `func TestGate_WorktreeLeftover(t *testing.T)`
- `plan:internal/fabricengine#SetLeftoverSeamForTest` -> `func SetLeftoverSeamForTest(t *Topology, beforeRemove func(), sleep func(time.Duration))`
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `internal/fabricengine/remove_leftover_integration_test.go`

**Edit:**
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverSeam) error`
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/fabricengine/doc.go`
- `pattern/PATTERN-fabric-destruction-chokepoint.md`

**Intent:**
D7: a task worktree removal finishes a directory `git worktree remove` left refilled, by a bounded retry of the leftover's contained removal.
When git cannot fully delete a linked worktree's directory it exits nonzero having already deleted the registration, and leaves the directory on disk (checked by hand against real git: a read-only subdirectory makes `git worktree remove` exit 255 with the worktree gone from `git worktree list` and the directory kept).
`removeWarpWorktreeDir` then finds the target unregistered and returns its not-a-linked-worktree error, so the `removePath` fallback is never reached.

The new executor lives in `internal/fabricengine/destroy.go`, the only file allowed a destructive primitive (`PATTERN-fabric-destruction-chokepoint`), and takes the recorder first (`PATTERN-mutation-record`).
`removeWorktreeLeftover(rec, req, gitErr, seam)` removes `req.target`, the directory git was asked to remove, under `req.container`, the hub:
- a nil `gitErr` is a fail-loud wiring guard error and removes nothing, since the executor's ownership proof is that git ran;
- containment is checked with `containmentFailure(req.container, req.target)`, refusing with a `destructiveRefusal` of `CheckContainment`, as the gate does;
- ownership and dirtiness are not re-probed: git ran on this target only after `removeGitWorktree`'s gate passed the same `req` (registered linked worktree, dirtiness of tracked and untracked files), and git had started deleting the work tree, so what is left is what a late writer put there;
- it removes the target recursively through `seam.remove` (nil means `removeContainedPath`); on a late-writer error it sleeps `leftoverRetryDelay` through `seam.sleep` (nil means `time.Sleep`) and removes again, at most `leftoverRetries` more times;
- on success it appends `KindPathRemoved` for the target with detail `recursive`, the entry `removePath` records for a directory, and nothing for an already-absent target;
- a failure after the last attempt, and any failure that is not a late-writer error, is returned at once, wrapped with the target and the attempt count.
`leftoverSeam` holds the `remove func(container, target string, recursive bool) (removed, wasDir bool, err error)` and `sleep func(time.Duration)` fields; its zero value is production.

`isLateWriterError` matches `err` with `errors.Is` against `lateWriterErrors`, which the per-OS files declare and which name no message text:
`latewriter_unix.go` (`//go:build !windows`) holds `syscall.ENOTEMPTY`;
`latewriter_windows.go` (`//go:build windows`) holds `windows.ERROR_DIR_NOT_EMPTY` and `windows.ERROR_SHARING_VIOLATION` from `golang.org/x/sys/windows`, already in `go.mod`, since Go opens Windows files without `FILE_SHARE_DELETE`, so a lock file its writer still holds open refuses deletion.
Neither file holds a destructive primitive, so the destructive guard scan passes.

`Topology` gains a `leftover leftoverSeam` field, zero in production like its `push` seam, and `Topology.Remove`, `removeWarpWorktreeDir`'s only caller, passes `t.leftover` through the new parameter.
`removeWarpWorktreeDir`'s branch after a `GitError` whose target is no longer a registered linked worktree calls `removeWorktreeLeftover(rec, req, gitErr, leftover)` when the target is still on disk, with `req` the request git ran under.
`RemoveWarpWorktreeDirForTest` in `export_test.go` keeps its four-argument type by becoming a function literal that calls `removeWarpWorktreeDir` with `leftoverSeam{}`, so its callers in `destructivegaps_integration_test.go` are unchanged.
`SetLeftoverSeamForTest`, in `export_test.go` (package `fabricengine`) beside `SetPushSeamForTest`, sets that seam on one `Topology`: its `remove` runs `beforeRemove` and then `removeContainedPath`, and its `sleep` is the one given; a nil argument keeps production for that part.
A successful leftover removal returns nil after the same best-effort `git worktree prune` the fallback runs.
A failed one returns today's message up to `it is not a linked worktree of this repo`, followed by `, and removing the directory git left failed: <error>` wrapping the executor's error, so a `destructiveRefusal` still reaches `errors.As`.
A target absent on disk returns today's error unchanged; a target git leaves still registered keeps today's `removePath` fallback; a target the gate refused before git ran is still refused by the first branch, which `TestRemove_RefusesForeignWorktreeWithoutDeletingIt` already pins.
Bound: only a late-writer failure is retried, only on the target git was asked to remove after the gate passed it, with an injected sleep and at most two retries, about four seconds; a writer outlasting that, such as a run-window launcher re-running `lyx reed attach` in a loop, fails the removal as today.

Tests.
`TestGate_WorktreeLeftover` (package `fabricengine`, untagged, in `destroy_test.go`) calls the executor over a temp hub holding a target directory with a file in it, a `req` declaring the hub as container, the target and `ownedRegisteredLinkedWorktree`, and a non-nil `*gitexec.GitError`, with a `seam` whose `sleep` records each delay.
Rows:
- `a late-writer error then a clean removal succeeds on the retry`: `seam.remove` fails once with `&os.PathError{Err: lateWriterErrors[0]}` and then calls `removeContainedPath`; want nil, the target gone, one sleep of `leftoverRetryDelay`, and `rec` holding exactly one `KindPathRemoved` entry for the target with detail `recursive`;
- `a persistent late-writer error is returned after the last attempt`: `seam.remove` always fails the same way; want an error matching `lateWriterErrors[0]` with `errors.Is`, three remove calls, two sleeps, the target on disk and `rec` empty;
- `another removal failure is not retried`: `seam.remove` fails once with `os.ErrPermission`; want that error, one remove call, no sleep and `rec` empty;
- `a target outside the container is refused`: a `req` whose target lies outside its container; want a `destructiveRefusal` of `CheckContainment` and no remove call;
- `a file its writer still holds open is removed once the writer closes it` (Windows only, skipped elsewhere): `seam.remove` nil, a file under the target held open, and `seam.sleep` closing it; want nil, the target gone and the `KindPathRemoved` entry.
The new `remove_leftover_integration_test.go` (`//go:build integration`, package `fabricengine_test`) holds the table-driven `TestRemove_GitLeftDirectory`.
Each row, over `hubforge.CopyHub`, `Add`s a pair, commits a file under a subdirectory of its task worktree, makes that subdirectory read-only so real `git worktree remove` exits nonzero with the registration deleted and the directory kept, and calls `Remove`; cleanup restores the mode, and the test skips on Windows and as root, where the mode stops nothing.
Rows:
- `a leftover the retry can remove finishes the removal`: `SetLeftoverSeamForTest` with a `beforeRemove` that restores the subdirectory's mode and a recording `sleep`; wants `Remove` to succeed, the target gone, `RemoveStepTaskWorktree` in `Steps`, and the result's mutation record holding a `KindPathRemoved` entry for the target with detail `recursive`;
- `a leftover whose removal fails for another reason is returned`: no seam; wants an error naming the target, holding `not a linked worktree of this repo` and the permission error, the target still on disk and no longer listed by `git worktree list` in the warp repo.
Both reach the new branch through real git; the permission error is not a late-writer error, so it is returned unretried.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/fabricengine` from 645 to 647 for the two new tests.
`destructiveGuardRecordingExecutors` gains `{"removeWorktreeLeftover", "func removeWorktreeLeftover(rec *Mutations, "}`.

Docs, in the same commit.
`internal/fabricengine/doc.go`'s `Remove` paragraph adds that a directory git left on disk after deleting the registration is removed by `removeWorktreeLeftover`, retried on a late-writer error.
`pattern/PATTERN-fabric-destruction-chokepoint.md` adds that the leftover removal after a partial `git worktree remove` takes its ownership and dirtiness from that git run's passed gate and checks containment itself.

**ImpactSummary:** `removeWarpWorktreeDir`'s unregistered-after-git-failure branch now removes the leftover directory instead of refusing; `Topology.Remove`, its only caller, passes a new zero-in-production seam, and every other branch is unchanged.
