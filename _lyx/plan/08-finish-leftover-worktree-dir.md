# Card 8 — finish-leftover-worktree-dir

**Create:**
- `internal/fabricengine/latewriter_unix.go`
- `internal/fabricengine/latewriter_windows.go`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#leftoverSeam` -> `type leftoverSeam struct`
- `plan:internal/fabricengine#removeLeftoverWorktreeDir` -> `func removeLeftoverWorktreeDir(rec *Mutations, req pathRequest, seam leftoverSeam) error`
- `plan:internal/fabricengine#ownedRemovedWorktreeLeftover` -> `func ownedRemovedWorktreeLeftover(repoDir, gitTarget string) pathOwnership`
- `plan:internal/fabricengine#pathOwnershipRemovedWorktreeLeftover` -> `const pathOwnershipRemovedWorktreeLeftover pathOwnershipKind = iota`
- `plan:internal/fabricengine#leftoverRetries` -> `const leftoverRetries = 2`
- `plan:internal/fabricengine#leftoverRetryDelay` -> `const leftoverRetryDelay = time.Second`
- `plan:internal/fabricengine#SetLeftoverSeamForTest` -> `func SetLeftoverSeamForTest(t *Topology, remove func(container, target string) (removed, wasDir bool, err error), sleep func(delay time.Duration))`
- `plan:internal/fabricengine#LeftoverRetryDelayForTest` -> `const LeftoverRetryDelayForTest = leftoverRetryDelay`
- `internal/fabricengine/remove_leftover_integration_test.go`
- `plan:internal/fabricengine_test#TestRemove_FinishesALeftoverTaskWorktreeDir` -> `func TestRemove_FinishesALeftoverTaskWorktreeDir(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#removeWarpWorktreeDir` -> `func removeWarpWorktreeDir(rec *Mutations, l *lyxcwd.Location, target string, force bool, leftover leftoverSeam) error`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#refusePrimeSlug`
- `internal/fabricengine#RemoveWarpWorktreeDirForTest`
- `internal/fabricengine/doc.go`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#removeContainedPath`
- `internal/fabricengine#dirtinessNA`

**Intent:** D7: when `git worktree remove` exits nonzero having already deleted the task worktree's registration and left its directory refilled by a late writer, `Remove` finishes the directory instead of returning the not-a-linked-worktree error.

`removeWarpWorktreeDir` gains a `leftover leftoverSeam` parameter.
In its branch after a `*gitexec.GitError` where the target is no longer a registered linked worktree (`!isRegisteredLinkedWorktree(l, target)`), it builds a `pathRequest` for the leftover (container `l.HubPath`, target `target`, ownership `ownedRemovedWorktreeLeftover(l.WorktreePath(), target)`, dirtiness `dirtinessNA` with the reason that `git worktree remove` already accepted discarding the work tree, `force` as passed) and calls `removeLeftoverWorktreeDir(rec, req, leftover)`.
On success it runs the best-effort `git worktree prune` as the fallback does and returns nil; on failure it returns today's not-a-linked-worktree error text with the removal's error added (`%w`), and a `*destructiveRefusal` propagates unwrapped as the fallback's does.
A target already gone from disk takes the same call, whose contained removal is the idempotent no-op every executor relies on: it records nothing and returns nil, so `Remove` goes on with the rest of the teardown rather than failing over a directory that no longer exists.
The earlier branches are untouched: a gate refusal before git ran still returns the refusal (`TestRemove_RefusesForeignWorktreeWithoutDeletingIt` keeps pinning it), a non-git error still destroys nothing, and a target git leaves still registered keeps today's gated fallback through `removePath`.
Update `removeWarpWorktreeDir`'s doc comment for the leftover branch, its ownership proof and its bound, and `refusePrimeSlug`'s doc comment, which says `removeWarpWorktreeDir`'s registered-worktree rule refuses to delete anything git declined to remove: it now deletes the leftover git itself unregistered.

The leftover keeps the four-check pipeline of `PATTERN-fabric-destruction-chokepoint`.
`ownedRemovedWorktreeLeftover` is a new ownership predicate, its kind `pathOwnershipRemovedWorktreeLeftover` appended to the `pathOwnershipKind` iota block, carrying `repoDir` and the path git was asked to remove in the existing `expectedTarget` field.
`resolvePathOwnership` gains its case: owned exactly when the target equals `expectedTarget` and is not a registered linked worktree of `repoDir`, with a reason naming the failed half otherwise.
This is D7's ownership proof, replacing the registered-linked-worktree check for this directory: git ran, so the gate's ownership and dirtiness checks passed for the target when this `Remove` began, git had accepted discarding the work tree's content, and the target is the path git was asked to remove.

`removeLeftoverWorktreeDir` lives in `destroy.go`, the only file permitted a destructive primitive, with the recorder threaded into it there.
It runs `checkPathRequest(req)` first, then the seam's remove, default `removeContainedPath(req.container, req.target, true)`, so containment is still enforced at the act; when that fails with an error `isLateWriterError` accepts, it calls the seam's sleep, default `time.Sleep`, with `leftoverRetryDelay` and removes again, at most `leftoverRetries` more times; any other error, and a late-writer error after the last attempt, is returned.
A successful removal appends `KindPathRemoved` for the target with detail `recursive`, the entry the gated `removePath` records for a directory, so `Remove`'s mutation record shows the deleted directory (`PATTERN-mutation-record`); an already-absent target records nothing.
What this lets through and what bounds it: it deletes only the one directory git was asked to remove after the gate passed it, only once git dropped its registration, retrying only a late-writer error with an injected sleep, at most twice.
`cmd/lyx/destructiveguard_test.go`'s `destructiveGuardRecordingExecutors` table, which lists every `destroy.go` executor that takes a leading `rec *Mutations`, gains the row `{"removeLeftoverWorktreeDir", "func removeLeftoverWorktreeDir(rec *Mutations, "}`.

`isLateWriterError` is matched with `errors.Is`, never on message text or git's stderr, in two per-OS files: `latewriter_unix.go` (`//go:build !windows`) accepts `syscall.ENOTEMPTY`; `latewriter_windows.go` (`//go:build windows`) accepts `windows.ERROR_DIR_NOT_EMPTY`, a refilled directory, and `windows.ERROR_SHARING_VIOLATION`, a file under the target still open by its writer, such as a reed lock file `internal/lock` holds, since Go opens Windows files without `FILE_SHARE_DELETE` (`golang.org/x/sys/windows`, already in `go.mod`).

`leftoverSeam` holds `remove func(container, target string) (removed, wasDir bool, err error)` and `sleep func(delay time.Duration)`; a nil field means production.
`Topology` gains a `leftover leftoverSeam` field (a struct field, so no arrow), zero in production, and `Topology.Remove` passes `t.leftover` to `removeWarpWorktreeDir`.
In `export_test.go`, `SetLeftoverSeamForTest` sets it, like `SetPushSeamForTest`, and `LeftoverRetryDelayForTest` exports the delay for the external test.
`RemoveWarpWorktreeDirForTest` becomes a function with today's four parameters that calls `removeWarpWorktreeDir` with a zero `leftoverSeam`, and its doc comment says it is that wrapper rather than a re-export, so its callers in `destructivegaps_integration_test.go` keep compiling unchanged.

`TestRemove_FinishesALeftoverTaskWorktreeDir`, in the new `integration`-tagged `remove_leftover_integration_test.go` of package `fabricengine_test`, is one table-driven test over `hubforge.CopyHub` and `hubforge.AddPairWith(..., fabricengine.AddOptions{SkipPush: true})`, driving `h.Topology.Remove(l, slug, false, false)` and reading `res.Mutated().Entries()`.
Each row commits the file it plants under the task worktree before `Remove`, so the pair is clean and `RemoveRefusal`'s no-force dirtiness check passes, and the gate lets git run.
The seam's sleep records each delay and never waits, except where a row says otherwise.
Each Linux and macOS row makes `git worktree remove` fail after it drops the registration: the committed file sits in a subdirectory the row then makes read-only (`0o555`), so git cannot unlink it and leaves the directory on disk, unregistered; a `t.Cleanup` makes it writable again.
These rows skip on Windows and when running as root, where the read-only directory would not stop git.
- "a late writer's leftover is removed on a retry": the seam's remove returns an `*os.PathError` wrapping `syscall.ENOTEMPTY` on its first call, and on its second makes the subdirectory writable and removes the target with `os.RemoveAll`, returning `(true, true, nil)`; `Remove` succeeds, the target is gone, exactly one sleep of `fabricengine.LeftoverRetryDelayForTest` was taken, and the record holds a `KindPathRemoved` entry for the target with detail `recursive`.
- "a persistent late-writer failure is returned after the last attempt": the seam's remove always returns the `ENOTEMPTY` error; `Remove` fails, its error says the target is not a linked worktree and wraps `syscall.ENOTEMPTY`, the seam's remove ran three times with two sleeps, and the target is still on disk.
- "any other removal failure is not retried": the seam's remove is nil, so the real contained removal hits the read-only subdirectory and fails with a permission error; `Remove` fails with that error wrapped, no sleep was taken, and the target is still on disk.
- "a sharing violation on Windows is retried once its writer closes the file", run only on Windows: the row commits a file under the task worktree and then holds it open, so `git worktree remove` cannot delete it and the first leftover removal fails with `ERROR_SHARING_VIOLATION`; the seam's remove is nil and its sleep closes the file, so the retry succeeds, `Remove` succeeds, the target is gone and the record holds the `KindPathRemoved` entry.
The new top-level test pushes `internal/fabricengine` past its row: raise it in `cmd/lyx/testdata/test-budget.yaml` from 645 to 646.

Docs: `internal/fabricengine/doc.go`'s `Remove` paragraph rewords its claim that `Remove` never deletes a directory git declined to remove unless it is a registered linked worktree: the one other case is the task worktree directory `git worktree remove` left on disk after dropping its registration, which the leftover predicate owns and the contained removal under the hub finishes, retrying a late-writer failure at most twice; a re-entered `Remove` otherwise reports the leftover as a stray path, as today.

**ImpactSummary:** fabricengine's task-worktree removal gains a bounded, fully gated leftover removal in destroy.go behind a new Topology seam and a re-signed private removeWarpWorktreeDir; its one production caller, the test exports, the destructive-guard table, the doc and the budget row follow.

**Commit:** 8: Finish a task worktree directory git worktree remove left refilled
