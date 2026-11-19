```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on a concrete input and checked for wrong results, swallowed errors and inconsistent state.

**Per function or branch, the case run and the verdict:**
- `oneRepositoryAt`, nothing at the path (`/hub/missing`): `Lstat(/hub/missing/.git)` returns ENOENT, so it answers false with no git, no log and no store. Correct.
- `oneRepositoryAt`, a directory with no `.git`: same as the missing path. Correct.
- `oneRepositoryAt`, an empty path `""`: the key becomes `"."` and is read against the cwd. Every caller passes an absolute worktree path, so this is no defect.
- `oneRepositoryAt`, a `.git` file pointing at a pruned gitdir: `Lstat` finds the file, then `dotgit.Read` fails on `requireDir` (`internal/dotgit/dotgit.go:46`). It logs and does not store, so it logs again on every call, as it did before the change. Correct.
- `oneRepositoryAt`, `Lstat` failing with EACCES: it falls through to the full read, which fails, logs and does not store. Correct.
- `oneRepositoryAt`, a nested non-root path: `dotgit.Read` already needed `.git` at the path itself (`internal/dotgit/dotgit.go:33`), so the `Lstat` short-circuit changes nothing here. Correct.
- `oneRepositoryAt`, a worktree read, then removed by fabric in the same process: the kept answer outlives the directory and still gives the hub's layout. Before the change this case logged and answered false. Same hub, same layout, so this is no defect.
- `oneRepositoryAt` against a concurrent `forgetLayouts`: goroutine A runs `readOneRepositoryLayout` with no lock held. Goroutine B commits a marker and calls `Clear`. A then calls `Store` with the answer it read before the commit. The stale answer stays for the rest of the process. **Finding 1.**
- `oneRepositoryAt` across processes: marker writes made by another process never clear this process's map. **Finding 2.**
- `readOneRepositoryLayout`, each error path (`dotgit.Read`, HEAD, `committedMarker` at HEAD, `BranchExists`, `committedMarker` on `lym-bolt`): each returns an error with its context, and the caller logs it once. The final `return found, err` returns false with the error, which matches the earlier behaviour. Correct.
- `readOneRepositoryLayout`, a sibling main worktree: answers false with a nil error, so the answer is stored. Correct.
- `forgetLayouts` in `commitAnchorMarker` (`anchormarker.go:85`): runs only when a commit landed. A no-op write commits nothing and changes no answer. Correct.
- `forgetLayouts` in `retireBoltAnchorMarker` (`anchormarker.go:211`): runs on success and on a skipped write, and does not run when `PullThenCommitWritten` errors. Clearing too often costs only a re-read. Correct, apart from the case in Finding 2.
- `forgetLayouts` in `seedOneRepositoryBoard`: runs only when the commit landed. Correct.
- `MergeStateOperation`, two-repository, `CHERRY_PICK_HEAD` with a clean index: `MergeHeadPresent` is false and `ConflictedFiles` is empty, so it answers `""`. Correct per the Goal.
- `MergeStateOperation`, two-repository, a cherry-pick stopped on a conflict: `ConflictedFiles` is not empty, so it answers `"merge"`. This matches the probe before f06acf39 (`git show f06acf39^:internal/fabricengine/mergestateactive.go`), which the Goal asks for. Correct.
- `MergeStateOperation`, two-repository, `MergeHeadPresent` failing: the error is wrapped and returned. Correct.
- `MergeStateOperation`, one-repository: the branch is unchanged and still goes through `recordsMergeKindAt` and `gitOperationAt`. Correct.
- `MergeStateActive`: returns `operation != ""` and passes errors through. Correct.

**Findings:**

1. **LOW, design.** `internal/fabricengine/onerepository.go:49-54`: a layout read can store an answer after a concurrent `forgetLayouts` has cleared the map, so the stale answer stays for the process.
   - Evidence: `oneRepositoryAt` reads git at line 49 and stores at line 54, holding no lock between the two. `forgetLayouts` (line 61) runs after the marker commit inside `commitAnchorMarker` (`anchormarker.go:79-85`), under the weft write lock, which the reader does not take.
   - Fabric runs reads side by side within one process; `add.go` runs its probes beside each other.
   - Fix: give the map a generation counter that `forgetLayouts` bumps, read it before the git read, and store only when it is unchanged.

2. **LOW, design.** `internal/fabricengine/onerepository.go:29-31`: the doc says "A hub's layout is fixed once its marker is committed, so an answer holds for the process", but retiring the hub-wide board marker changes the Bolt checkout's answer.
   - Fabric clears the map only in the process that makes the write. A long-lived process (`loomcli/start.go`, `orchcli/watch.go`, `creelcli/delivery.go` all call fabric) keeps the old answer after another process's `lym fabric reconcile` retires the marker (`anchormarker.go:197-211`).
   - The lock directory `recordsLockDir` picks then differs between processes, so two processes no longer take the same lock.
   - The Goal asks for a cache "for the process", so the scope itself is licensed. What is wrong is that the doc states an invariant the retirement path breaks.
   - Fix: state the bound in the doc, or skip the cache for the Bolt checkout and the main worktree, whose answer depends on the `lym-bolt` branch.

**Tool calls:**
- Grep of clone and reconcile and `tendercli/runintents.go`: to check which function the clone and reconcile marker commits go through, and whether tender's probe call sits in a daemon. My context lacked both.
- Grep for long-lived fabric callers: to find which processes outlive a marker write. My context lacked it.
- Read of `dotgit.Read`: to compare its `Stat` with the new `Lstat` check and its handling of a nested path. My context lacked it.
