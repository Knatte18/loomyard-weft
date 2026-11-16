```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

**MEDIUM: under F1, a crash during a re-entry attempt resets the retry allowance, so "a crash never resets it" is false.**
- Changed text:
  - Card 12, `runBatch` bullet: "counts the lost attempts on the rung from the batch's latest `Reentry` attempt on … The count is read from recorded attempts, never from a counter of its own call, so a crash and a new call never reset it."
  - Card 13, begin bullet: "with `reentry` set when a `reentry` names the batch, so its first attempt is marked `Reentry`."
- Concrete case:
  1. A batch is stopped and re-entered. Its re-entry attempt (marked `Reentry`) is in flight when the runner crashes.
  2. Card 13 line 56 clears `StopRecord` only when a batch is recorded done or a verify re-run passes. It says this holds "also when a re-entered attempt ends done after `reenter` has returned", so `StopRecord` stays set for the whole re-entry attempt.
  3. On the next call, `resumeStrands` judges the in-flight attempt lost. Then, because `StopRecord` is still set, `Run` calls `reenter` again (card 13 line 29). `reenter` names the batch, so `runBatch` marks a second `Reentry` attempt and the lost count starts again from it.
  4. Each crash therefore grants a fresh same-rung retry. Before the fix, the cumulative count bounded this.
- Evidence: card 13 lines 29 and 56; card 12's new `runBatch` bullet.
- Possible fix: mark `Reentry` only on the first re-entry of a given stop. One way is to record on `StopRecord` that it has been re-entered. Another is to skip the mark when an attempt of the batch already began after the stop was recorded. The same applies to the verify re-entry's `Reentry` fixer.

**MEDIUM: under F5, the new owner sentence leaves stops judged by the resume without a named writer.**
- Changed text: card 12, "`runBatch` sets it for a batch or needs-parent stop, `Run` for a verify stop."
- Concrete case: card 13 `resumeStrands` step 4 judges a crashed batch's last attempt itself: "a failure on the last rung, or a `needs_parent` attempt, stops the row as `runBatch` does." That is a batch or needs-parent stop set and returned on the resume path, outside `runBatch`.
- Two of the new card 12 sentences contradict each other there:
  - The general sentence says every batch and needs-parent stop sets `Stop` before Stuck.
  - The owner clause names only `runBatch`.
- This is the review's own premise (which code sets it). It reappears for the resume path, and no `TestRun_Resume` row asserts `Stop` after a resume-judged last-rung failure.
- Evidence: card 13 step 4; the new card 12 sentence.
- Possible fix: name `resumeStrands` as a writer for the stops it judges, and add the matching `TestRun_Resume` assertion.

**Checked without a finding:**
- **F2:** removing `outcome.yaml` and `summary.md` does not dirty the task worktree's clean-tree check. `checkCleanTree` uses `git status` on the code worktree, and the run directory's writes, `state.json` included, already happen between commits without failing an attempt.
- **F3 and F4:** I found no breaking case.

Tool calls:
1. Grepped card 10 for `checkCleanTree` and `commitRecords`, and card 14 for `archiveRunInPlace`. My context lacked how the clean-tree check and the archive treat the run files that F2 removes.
2. Grepped `DirtyPaths` and `verifytree.DirtyPaths`. My context lacked whether the clean-tree check would see the deleted run files.
