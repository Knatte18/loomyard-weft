# Card 5 — Finish the task worktree directory `git worktree remove` left behind

**Create:**
- `plan:internal/fabricengine#leftoverRemovalRetries` -> `const leftoverRemovalRetries = 2`
- `plan:internal/fabricengine#leftoverRetryDelay` -> `const leftoverRetryDelay = time.Second`
- `plan:internal/fabricengine#isLateWriterError` -> `func isLateWriterError(err error) bool`
- `plan:internal/fabricengine#removeGitWorktreeFinishingLeftover` -> `func removeGitWorktreeFinishingLeftover(rec *Mutations, req pathRequest, repoDir string, remove func(container, target string, recursive bool) (removed, wasDir bool, err error), sleep func(time.Duration)) (leftoverErr, err error)`
- `plan:internal/fabricengine#removeLeftoverWorktreeDir` -> `func removeLeftoverWorktreeDir(rec *Mutations, container, target string, remove func(container, target string, recursive bool) (removed, wasDir bool, err error), sleep func(time.Duration)) error`
- `plan:internal/fabricengine#lateWriterErrnos` -> `func lateWriterErrnos() []error`
- `plan:internal/fabricengine#TestRemoveLeftoverWorktreeDir` -> `func TestRemoveLeftoverWorktreeDir(t *testing.T)`
- `plan:internal/fabricengine#RemoveGitWorktreeFinishingLeftoverForTest` -> `var RemoveGitWorktreeFinishingLeftoverForTest = removeGitWorktreeFinishingLeftover`
- `plan:internal/fabricengine_test#TestRemoveWarpWorktreeDir_LeftoverAfterGitFailure` -> `func TestRemoveWarpWorktreeDir_LeftoverAfterGitFailure(t *testing.T)`

**Edit:**
- `internal/fabricengine#removeWarpWorktreeDir`
- `internal/fabricengine#KindPathRemoved`
- `internal/fabricengine/doc.go`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D7, under `PATTERN-fabric-destruction-chokepoint`, `PATTERN-pair-teardown` and `PATTERN-mutation-record`.
Premise: when git cannot fully delete a linked worktree's directory, `git worktree remove` exits nonzero having already deleted the registration, so `removeWarpWorktreeDir` finds the target unregistered and returns its not-a-linked-worktree error, and the leftover directory stays.

`internal/fabricengine/destroy.go` (the only file allowed the act):
- `removeGitWorktreeFinishingLeftover` is the executor `removeWarpWorktreeDir` calls in place of `removeGitWorktree`.
  It calls `removeGitWorktree(rec, req, repoDir)`, so the pipeline (containment, the registered-linked-worktree ownership `req` declares, dirtiness, force) runs before git exactly as today.
  It returns that error unchanged as `err` with a nil `leftoverErr` unless all hold: the error is a `*gitexec.GitError` (git ran), `isRegisteredLinkedWorktreeIn(repoDir, req.target)` is now false, and the target is still on disk.
  Then it calls `removeLeftoverWorktreeDir(rec, req.container, req.target, remove, sleep)`; on success it returns `(nil, nil)`, otherwise `(that removal error, the git error)`.
  A nil `remove` means `removeContainedPath` with `recursive` true, so no file but `destroy.go` names the removal; the seam exists so a test can script a late writer over a real git failure.
  Ownership proof for this leftover, replacing the registration check git's own delete has made false: the gate passed the target as a registered linked worktree and clean (or forced) in this same call, git ran on exactly this target and had already accepted discarding its content, so what is left is what a late writer put there.
  A gate refusal is a `*destructiveRefusal`, not a `GitError`, so a target the gate refused before git ran never reaches the leftover removal.
- `removeLeftoverWorktreeDir` removes `target` under `container` through `remove` (production passes `removeContainedPath` with `recursive` true, the `os.Root` removal under the hub).
  A late-writer failure (`isLateWriterError`) is retried after `sleep(leftoverRetryDelay)`, at most `leftoverRemovalRetries` more times; any other failure, or a late-writer failure after the last attempt, is returned at once.
  A successful removal of an entry appends `KindPathRemoved` for `target` with detail `recursive`, the entry `removePath` records for a directory; an already-absent target records nothing.
  A nil `sleep` sleeps with `time.Sleep`.
- Both take `rec *Mutations` first, so `destructiveGuardRecordingExecutors` in `cmd/lyx/destructiveguard_test.go` gains a row for each (`"func removeGitWorktreeFinishingLeftover(rec *Mutations, "`, `"func removeLeftoverWorktreeDir(rec *Mutations, "`).
  Neither spells a banned token of the destructive guard outside `destroy.go`.

`isLateWriterError` lives in two per-OS files and reads no message text: `internal/fabricengine/latewriter_unix.go` (`//go:build !windows`) answers `errors.Is(err, syscall.ENOTEMPTY)`;
`internal/fabricengine/latewriter_windows.go` answers `errors.Is` against `windows.ERROR_DIR_NOT_EMPTY` (a refilled directory) or `windows.ERROR_SHARING_VIOLATION` (a file under the target still open by its writer, such as a reed lock file `internal/lock` holds, since Go opens Windows files without `FILE_SHARE_DELETE`), from `golang.org/x/sys/windows`, already a module dependency.

`KindPathRemoved`'s comment in `internal/fabricengine/mutation.go` names both writers: `removePath`'s deletion and the leftover worktree directory `removeLeftoverWorktreeDir` finishes.

`removeWarpWorktreeDir` (`internal/fabricengine/remove.go`) calls `removeGitWorktreeFinishingLeftover(rec, req, l.WorktreePath(), nil, nil)`; nil sleep means the real one, since the late writers are short-lived processes started as the reed session closed.
On a nil `err` it returns nil as today.
In its existing after-`GitError` not-registered branch, a non-nil `leftoverErr` is added to the returned not-a-linked-worktree message as "; removing the directory git left failed: <leftoverErr>", wrapped with `%w` so `errors.Is` still finds the removal error.
A target git leaves still registered keeps today's `removePath` fallback, and the gate-refused-before-git branch is unchanged.
Bound: only a late-writer failure is retried, only on the target git was asked to remove after the gate passed it, at most two extra attempts one `leftoverRetryDelay` apart; a re-entered `Remove`'s stray-path handling is untouched.

Tests:
- `lateWriterErrnos` is a test helper in two per-OS test files of package `fabricengine`, `internal/fabricengine/latewriter_unix_test.go` (`//go:build !windows`, answers `syscall.ENOTEMPTY`) and `internal/fabricengine/latewriter_windows_test.go` (answers `windows.ERROR_DIR_NOT_EMPTY` and `windows.ERROR_SHARING_VIOLATION`), so one table reaches every late-writer value of the OS it runs on.
- `TestRemoveLeftoverWorktreeDir` (Tier 1, new untagged file `internal/fabricengine/remove_leftover_test.go`, package `fabricengine`) drives `removeLeftoverWorktreeDir` with a scripted fake `remove` and a recording `sleep`, over a `NewMutations("")` record.
  Rows: removed on the first attempt (nil, one `KindPathRemoved` entry for the target with detail `recursive`, no sleep); for each `lateWriterErrnos()` value, a `*os.PathError` wrapping it then a removal (nil, one entry, one sleep of `leftoverRetryDelay`); the same value on every attempt (an error for which `errors.Is` finds it, exactly `1+leftoverRemovalRetries` attempts, `leftoverRemovalRetries` sleeps, no entry); a `*os.PathError` wrapping `fs.ErrPermission` (returned after one attempt, no sleep, no entry); already absent (nil, no entry).
  One more row, skipped off Windows, passes the real `removeContainedPath` over a `t.TempDir()` container whose target directory holds a file kept open with `os.Open`: the first attempt fails with a sharing violation, the `sleep` seam closes the file, and the call returns nil with the target gone, one sleep and one `KindPathRemoved` entry — the Windows criterion.
- `RemoveGitWorktreeFinishingLeftoverForTest` in `internal/fabricengine/export_test.go` re-exports the executor for the integration test, beside `RemoveWarpWorktreeDirForTest`.
- `TestRemoveWarpWorktreeDir_LeftoverAfterGitFailure` (Tier 2, added to `internal/fabricengine/destructivegaps_integration_test.go`, `//go:build integration`, package `fabricengine_test`; skipped on Windows and as root) is a table over a pair added with `hubforge.AddPairWith`, whose task worktree holds a subdirectory with a file set to mode `0o500`, so `git worktree remove --force` exits nonzero after deleting the registration and leaves the directory; cleanup restores the mode.
  Row `a late writer then a clean removal`: calls `RemoveGitWorktreeFinishingLeftoverForTest` with a `remove` that answers a `*os.PathError` wrapping `lateWriterErrnos()[0]` once, then restores the subdirectory's mode, removes the target with `os.RemoveAll` and answers removed as a directory, and a recording `sleep`; it asserts `(nil, nil)`, the target gone, `fabricengine.List` not listing it, one sleep, and a record holding one `KindPathRemoved` entry for the target with detail `recursive`.
  Row `a permission failure is not retried`: calls `RemoveWarpWorktreeDirForTest(rec, l, target, true)` with the production seams; the leftover removal fails with `EACCES`; it asserts the error names "not a linked worktree of this repo" and wraps `fs.ErrPermission`, the target still on disk, `fabricengine.List` not listing it, and no `KindPathRemoved` entry.
  It is not a row of `TestRemoveWarpWorktreeDir_FallbackHonoursForce`, whose rows reach a target git leaves registered, a state this table never reaches.
- The gate refusing before git ran stays pinned by `TestRemove_RefusesForeignWorktreeWithoutDeletingIt`, unchanged.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/fabricengine` from 645 to 647.

`internal/fabricengine/doc.go`: the `Remove` paragraph adds that a removal whose `git worktree remove` failed after unregistering the target finishes the leftover directory under the hub, retrying a late writer's not-empty or (on Windows) sharing-violation failure twice a second apart, and records it as `path_removed`.

**ImpactSummary:** One mutation-kind comment, one test export, and one teardown path in fabricengine (removeWarpWorktreeDir, reached only from Topology.Remove and its test export) gains a bounded leftover removal inside destroy.go; removeGitWorktree's own signature and its other callers (add, prune, weft wiring) are untouched.

**Commit:** 5: fabric: finish a task worktree directory git left refilled
