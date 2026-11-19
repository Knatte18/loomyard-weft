```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (`oneRepositoryAt` and `forgetLayouts` in `onerepository.go`, and the new `TestOneRepositoryAt_KeepsAnswerPerWorktree`) through the fix-code-correctness lens.

**Findings**

1. **LOW** — `onerepository.go`, the `layoutClears.Load() == clearsBeforeRead` guard before `layoutAnswers.Store`.
   - **Summary:** The guard narrows the stale-store window but does not close it, so the F2 premise can still happen.
   - **Evidence:** The check and the `Store` are two separate steps. Take this interleaving:
     - The reader's second `Load` equals `clearsBeforeRead`.
     - `forgetLayouts` then runs `Add(1)` and `Clear()` in full.
     - The reader then runs `Store(key, preCommitAnswer)`.
     - The stale answer survives the clear, which is the defect the review described.
   - **Doc claim:** The new doc line "A read that overlaps a clear is not kept" overstates the guarantee.
   - **Fix:** Soften the wording, or close the window with a mutex held across check-and-store and `forgetLayouts`.
   - The reviewer's own proposed fix has the same gap, and the window is narrower than before.

2. **LOW** — `mergestateactive.go`, line 13, the rewritten comment ("a cherry-pick, revert or rebase there does not count, since fabric's own weft replay cherry-picks under the write lock the commit waits on.").
   - **Summary:** The line is about 140 characters of text, over the 120-character limit in the prose skill's line-break rule.
   - **Evidence:** It is one unbroken line. Break it at the comma before "since".

**Traced and correct**

- **Normal read path:** `clearsBeforeRead` is read before `readOneRepositoryLayout`. On a failed read the function returns false without storing, so failures are still not kept.
- **Clear ordering:** `forgetLayouts` increments the counter before `Clear()`, so a reader that stores between the two has its entry removed by the `Clear`.
- **Callers:** `forgetLayouts` keeps its signature, so its callers (`anchormarker.go:85`, `:211`, `clone_onerepository.go:293`) are unaffected.
- **`KeptAcrossAnOutsideMarkerRemoval`:**
  - Marker on HEAD gives true on the first read.
  - After the marker is removed and `lym-bolt` is deleted, a fresh read gives false (no marker at HEAD, `prime` is the main worktree, `BranchExists` is false).
  - So the row fails if `Store` is dropped.
- **`DroppedByAFabricMarkerCommit`:**
  - The first read is false and is kept.
  - `commitAnchorMarker` commits the marker, so a fresh read gives true.
  - The row fails if `forgetLayouts` is emptied or its call in `commitAnchorMarker` is removed.
- **`FailedReadNotKept`:**
  - The empty `.git` file passes the `Lstat` check, so `dotgit.Read` runs, fails and logs a Warn.
  - After the repair the next read gives true.
  - The row fails if a failed read were stored.
- **Parallelism:** The test is serial and calls no `t.Parallel`, so parallel tests stay paused while it runs. No other test can clear the shared map during a row.
- **Not covered:** No test exercises the new clear-counter guard (F2). The review did not ask for one, so I do not count that as a finding.

**Tool calls:** none. The context already held every changed file and the review.
