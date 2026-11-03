```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| Task worktree refilled after git's clean exit is removed (`removeWarpWorktreeDir` → `finishGitWorktreeRemoval`) | `TestRemove_FinishesWorktreeDirGitLeftOnDisk`, row "a task worktree refilled after git's clean exit is removed". It asserts the directory is gone, a `path_removed` mutation, and `RemoveStepTaskWorktree`. |
| Sibling worktree refilled after a clean exit, non-local path (`removeWeftWorktree`) | Row "a sibling worktree refilled after git's clean exit is removed". |
| Local-only pair's sibling refilled after a clean exit (`removeLocalWeft`) | Row "a local pair's sibling worktree refilled…". |
| Sibling that git deregistered but failed to delete is finished (nonzero `GitError`, not registered, still on disk), via `removeWeftWorktree` | Row "a sibling worktree git deregistered and failed to delete is removed". Skipped on Windows and as root. |
| Finish fails after a clean exit: error wraps the cause and the directory stays (task side) | Row "a refilled task worktree whose removal fails…". |
| `refill` seam is nil in production, and `SetLeftoverSeamForTest` now assigns fields | The existing `destructivegaps_integration_test.go` and `remove_guard_integration_test.go` rows still use the seam setter. The new rows exercise the refill field. Nothing else is needed. |
| `rollbackAdd` now passes `t.leftover` to `removeWeftWorktree`, so Add's rollback finishes a leftover weft directory | None. See finding 1. |
| Gate refusal or non-`GitError` returns `gitErr` unchanged | Existing: `remove_guard_integration_test.go:82` asserts the seam is not run on a gate refusal, and the sibling-dirty and refusal-remedy tests cover refusals. No new row, and none is needed. |
| `GitError` while the target is still registered returns `gitErr` and deletes nothing, on the sibling side | None in the diff. See finding 2. |
| Nonzero git with nothing on disk returns `gitErr` unchanged | None new. |
| Sibling finish fails after git deregistered it: wrapped "deregistered it and failed (…)" error, then `removeSibling` / `removeLocalWeft` report "weft worktree remains" | None. See finding 3. |
| Steps reported for a finished sibling | Rows assert `wantStep`. The local row is weak, because `removeLocalWeft` appends the step whenever `hadWorktree` was true. |

**Findings**

1. **MEDIUM, design (coverage gap).** `internal/fabricengine/add.go:852`, `rollbackAdd`. The diff adds the behaviour that Add's rollback finishes a leftover weft directory, and the summary claims it. No test row drives a failed Add whose weft directory is refilled. The new test only calls `Topology.Remove`. Evidence: `remove_leftoverdir_integration_test.go:113` calls only `h.Topology.Remove`. The existing rollback tests (`add_rollback_adopt_test.go:120`, `origin_integration_test.go:269`) never set the refill hook.

2. **MEDIUM, design (coverage gap).** `destroy.go:1409-1411`, the `isRegisteredLinkedWorktreeIn(repoDir, target)` guard in `finishGitWorktreeRemoval`. This is the ownership safeguard that stops the sibling side from deleting a directory git refused to remove while it was still registered, such as a locked worktree. No row reaches it from the sibling or local-weft callers. Row 4 only covers the deregistered case. If the guard were dropped, no test in the diff would fail. Existing task-side tests exercise `removeWarpWorktreeDir`'s own branch, not the new helper's guard.

3. **LOW, design (coverage gap).** `destroy.go:1417-1420` and `remove.go:469-472`. The error path where git failed and deregistered the directory and the finish also fails has no row. The `gitErr != nil` wrapper is unasserted, and neither is the sibling-side "weft worktree remains" outcome. Row 5 covers only the task side after a clean exit. Sibling-side failure is untested in both `removeSibling` and `removeLocalWeft`.

**Test economy**
- No redundancy found. Rows 2 and 3 reach different callers (`removeWeftWorktree` and `removeLocalWeft`). Row 4 is the only sibling coverage of the nonzero path. The task-side read-only row already exists in `destructivegaps_integration_test.go`, and the new read-only row is sibling-side only.
- The read-only row's guards are acceptable. Windows generates no subtest, the root skip names a prerequisite, and `t.Cleanup` restores permissions.

**Tool calls:** none. My inherited context already held everything this lens needed.
