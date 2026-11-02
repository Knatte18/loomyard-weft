```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the round-1 diff (`internal/landingshed` lock handling and `internal/parentlock` queue and wake). I made no tool calls. I judged each case from the code already in context and did not run any of it.

**Per function or branch: case run, result**
- `Call` loop and `landUnderLock`, no lock store: `freeLock` returns early, `lockFreed` stays false, one pass. Correct.
- `landUnderLock`, red verify → fixer → let through: `freeLock` releases and sets `lockFreed`. `mergeAndGate` returns done=false straight from `judgeFixerChange`, so `requeue` runs before `commitParentRecords`, `recheckLock`, `mergeOnGitHub` and the parent-side merge. Correct. No exit frees the lock and then lands without it.
- `landUnderLock`, fixer errors or the context is cancelled after `freeLock`: the error is returned with no landing, and the deferred `release` is a no-op because the record is gone (`Release` checks `SameProcess`). Correct.
- Merge-in-required retry (`finalize.go:324-329`): the same `lockFreed` check follows the second `mergeAndGate`. Correct.
- `judgeFixerChange` review loop: `freeLock` runs once. A rejection goes to `runFixer`, where `freeLock` is a no-op, and the result is judged again. Correct. Each requeue spends bounded `Spent` or `ReviewRounds`, so the pass loop terminates.
- Requeue pass, tree verified: `gateAndFix` always runs `gate.check` first, so a merge that brought parent commits is verified under the lock. Only a tree identical to the verified one is skipped. A tree that differs from the verified one cannot land unverified. Correct.
- Second failed verify: `rec.Spent` persists in the attempt record, so `spent` ends the run and no fresh cap is granted. Correct.
- `rebaseAfterParentMerge`, `Judged` set and a merge onto `Judged`: Baseline and `Judged` move to the merged head. A later fixer is measured from the merge (matches the `t2..f1,f1..f2` test). A retry merge inside the same pass moves them again. Correct.
- `rebaseAfterParentMerge`, no-op merge or `merge.TaskHead != Judged`: the record is left alone. Correct for the cases that set `Judged`. See F1 for the case that does not.
- `judgeFixerChange` with `Baseline == head`: proceeds without recording `Judged`. Wrong, see F1.
- `tryAcquire` with `front` set: it passes non-front tickets, waits behind an older uncovered front ticket and is granted past its own ticket. A non-front acquirer waits behind front tickets. Correct. `keepTicket` writes `Front` only on the first write, which is fine because a ticket belongs to one waiter. A cancelled, paused or errored wait removes the front ticket through `abandonTicket`, and a grant removes it. A dead holder's ticket is pruned in `liveTickets`. Correct.
- `Acquire` watch, release before the watch opens: the first poll delay of one second covers it. Correct.
- `Acquire` watch, release between attempt and `pause`: `fswatch` forwards on an unbuffered channel, so the pending event is delivered when `pause` selects. Correct.
- `Acquire` watch failure: logs a Warn once and polls on. Correct.
- `pause`, change arrives: the sleep is cancelled, the floor is waited out from `wokeAt`, and the goroutine is always drained. Correct.
- `pause`, `changes` closed, only if the fsnotify channel closes: a closed channel fires every iteration, so the wait wakes every second and never backs off to the ceiling. See F2.
- `pause`, no events: the timer bounds the wait at the poll ceiling. Correct.
- `ensure` at the front: OnWait, the wait record and the mark are set and removed as before. Correct.

**Findings**

- **F1, MEDIUM, design, `internal/landingshed/finalize.go:820-822` (with `rebaseAfterParentMerge`, 717-734).**
  - Summary: a fixer turn that leaves HEAD at Baseline returns `proceed()` without recording `Judged`, so a later merge of parent commits is never rebased and the parent's lines count as the fixer's change.
  - Case: the first red verify makes `runFixer` set Baseline to HEAD. The fixer fixes it without a commit (a flaky or environmental failure, or an edit it reverted), so `judgeFixerChange` hits `Baseline == head` and `Judged` stays empty. After requeue the merge brings parent commits. `rebaseAfterParentMerge` returns early because `rec.Judged == ""`. If the verify then passes, `LineChange(Baseline, head)` counts the parent's commits. Over the caps that spuriously triggers fix reviews or a halt, which is the defect the Judged and Baseline move was added to prevent.
  - Fix: in the `Baseline == head` branch, set `rec.Judged = head` and write the record before proceeding, as `letThrough` does. Alternatively, have `rebaseAfterParentMerge` also rebase when `merge.TaskHead == rec.Baseline` and the merge was not up to date.

- **F2, LOW, design, `internal/parentlock/acquire.go:76`.**
  - Summary: `pause` does not check whether `changes` was closed, so a watcher whose events channel closes mid-wait turns the poll into a one-second spin and the backoff to the thirty-second ceiling is lost.
  - Case: the `<-changes` receive returns the zero value at once on a closed channel, and every pause then costs only the remaining floor.
  - Fix: use `case _, ok := <-changes` and, when `!ok`, return `w.sleep(ctx, d)`. In `Acquire`, set `changes = nil` for later iterations, for example by having `pause` report the closed channel.

- **F3, LOW, design, `internal/landingshed/finalize.go:215`.**
  - Summary: the front priority is lost when the requeue wait halts at a pause. A resume goes through `Call` with `front=false`, so the returning run queues at the back, though the decision gives a run back from its fixer the head of the queue.
  - Case: after a fixer, the requeue wait returns the pause reason as a halting Stuck, and the resume restarts at `front := false`.
  - Fix: let a resumed Finalize queue at the front when its attempt record shows a fixer change let through (`Judged != ""`), or document the bound. Judge whether to fix or only document it.

**Tool calls:** none.
