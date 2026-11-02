# Finalize frees the parent lock while its fixer works

Finalize held the parent lock from its merge of the parent through the verify, and on a red verify through the whole fixer session and the re-verify.
One run whose verify failed could hold the lock on `main` for many minutes while green runs queued behind it.

## What changes

**Finalize (`internal/landingshed`).**
Finalize still takes the lock before it merges the parent and verifies, so a run that verifies green lands what it verified with no second merge.
When the verify fails, Finalize frees the lock before the merge fixer starts, and before any fix review session, since neither touches the parent.
Once the fixer's change is let through, Finalize takes the lock again and runs every step under the lock again, from the pull-request step on: it merges the parent anew and verifies under the lock as on its first try.
When nothing landed on the parent meanwhile, that merge is a no-op and the verify is skipped over the tree the fixer already verified.
`Call` now loops over `landUnderLock`, one locked pass, which reports when it freed the lock and wants another pass.
A Finalize with no lock store frees nothing and goes on in the same pass, as before.

**Head of the queue (`internal/parentlock`).**
A run taking the lock back queues at the front, ahead of runs that queued while it fixed.
A `Waiter` with `Front` set writes a ticket marked front; the queue orders front tickets first, then the rest, each oldest first.
`parentGuard.ensure` takes the front flag; Parent-Align passes false.

**Hand-over latency (`internal/parentlock`).**
`Acquire` watches the branch's lock record through `internal/fswatch`, so a release ends the poll delay at once, still a second after the last attempt began, instead of up to thirty seconds later.
A watch that cannot open logs a Warn and leaves the wait on its poll.

**Recovery ladder.**
A failed verify on a pass after the lock was taken again spawns the fixer on the attempts `finalize_fix_attempts` leaves the entry, never a fresh cap, so the passes end within that cap.

**Measuring the fixer's change across a merge.**
The fixer's change was measured as `LineChange(Baseline, HEAD)`, which counts a merge of the parent made after the fixer as the fixer's own lines; the old merge-in-required retry already had this flaw, and with the lock freed the parent now often moves during the fix.
The attempt record gains `judged`, the HEAD at which the fixer's change was last let through (within the caps, approved or accepted).
A pass whose merge brings parent commits onto that HEAD moves the entry's `Baseline` to the merged head, so the parent's commits never count as the fixer's change.
`judgeFixerChange` treats a Baseline equal to HEAD as no change.

## Design decisions

- **Wait mark and `lym loom status` while fixing outside the lock.** No new mark: while the fixer works, status shows its session as it does today, and while the run queues again it shows the existing parent lock wait (`Finalize: waiting for the parent lock on <branch> held by <slug>`). An Info log line records each freeing of the lock.
- **Fix reviews also run with the lock freed.** The record names only the fixer, but a fix review is an agent session on the task worktree alone, just as long, so it frees the lock too.
- **Wake by file event rather than a lower poll ceiling.** PATTERN-daemon-wakeups asks for a file event wherever the substrate offers one, and the PR-Review wait already wakes this way. The thirty-second ceiling stays for the case where the watch cannot open.
- **Bound.** A fixer change let through before a merge of parent commits and one made after it are each judged against the caps alone, not summed.

## Tests

- `internal/parentlock`: a release ends the poll delay after the floor; a front waiter is granted ahead of an older ticket, which stays queued.
- `internal/landingshed`: the fixer runs with the lock freed and the run merges and verifies again before it lands, with a second failed verify spending the remaining attempt and its change measured from the merge; `ensure` at the front is granted ahead of an earlier ticket.
- The lock re-check tests released the lock in the fixer's turn, which now runs outside the lock; they release it in the parent-records commit instead, which still sits between the acquire and the re-check.

## Docs

`internal/landingshed` and `internal/parentlock` package docs, `docs/overview.md`'s Finalize line, and PATTERN-daemon-wakeups' entry for the parent lock wait.
