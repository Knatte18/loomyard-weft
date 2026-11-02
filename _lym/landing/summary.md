# Finalize frees the parent lock while its fixer works

Finalize used to hold the parent lock through its whole row: verify, and on a red verify the fixer session and the re-verify.
One run held `main` for 12 minutes that way while four green runs queued behind it.
Finalize now frees the lock before its merge fixer or a fix review starts, since neither touches the parent, so queued runs land meanwhile.

## What changes

- Finalize still takes the lock before its merge and verify, so a run that verifies green lands what it verified with no second merge.
- On a red verify, or a fix review over a passing gate, `Finalize` frees the lock before the agent session starts.
  Once the fixer's change is let through, it takes the lock again, merges the parent anew and verifies under the lock before it lands.
  The merge is a no-op and the verify is skipped when nothing landed meanwhile.
- The returning run queues at the front, ahead of runs that queued while it fixed.
  `parentlock` tickets carry a `Front` flag, and the queue order is front tickets first, then the rest, each oldest first.
  A `Finalize` resumed with an attempt record showing a fixer change let through queues at the front too.
- A queued run no longer waits out the poll delay after a release.
  `parentlock.Acquire` watches the branch's lock record through `internal/fswatch`, and a change ends the delay early, still a second after the last attempt began.
  A watch that cannot open or closes mid-wait leaves the wait on its poll backoff, and logs one warning on open failure.
  This addresses the 21 to 23 second hand-overs seen on 2026-11-01.
- `lym loom status` shows the fixer's session while it works and the parent lock wait while the run queues again.
- A hub pause that covers the run holds its fixer's gate calls and its requeue wait like any other run's, since the run holds no lock then.
  The lock-holder exemption in `loomcli/landingdeps.go` stays as it is.

## Design decisions

- The lock is not taken only at the very end: the parent moves between the run's merge and its landing, so the run would merge, verify and fix again and could starve while much lands.
- `Finalize.Call` runs `landUnderLock` in a loop, once per locked pass, and a pass that freed the lock returns to Call to run again.
  A Finalize with no lock store frees nothing and goes on in the same pass.
- A pass after the lock is retaken spends from the same fixer attempts, so a second failed verify never grants a fresh cap.
- The attempt record gains `Judged`, the HEAD at which the fixer's change was last let through.
  When a later merge brings parent commits onto that HEAD, the entry's Baseline moves to the merged head, so the parent's commits never count as the fixer's change.
  A fixer change let through before such a merge and one made after it are each judged against the caps alone, so their sum can pass the cap unseen.
- Finalize's retry after a merge-in-required error also frees the lock if a fixer runs there, and retakes it before it lands.
- `parentguard.ensure` takes a `front` argument; `ParentAlign` passes false.

## To check by hand

- Not run: a real hub with a red verify, to see the lock freed during the fixer and a queued run landing meanwhile.
  The tests drive the lock store and the fixer's turn with fakes.
- Finding: the fix review running with the lock freed is pinned only through the approval path, in `internal/landingshed/finalize_gate_test.go`; check that a rejection and the fixer's turn after it also run with the lock freed.
- Finding: a fixer turn that ends with no commit leaves `Judged` empty or set in `judgeFixerChange` in `internal/landingshed/finalize.go`; check that the run never reads the next parent merge as the fixer's change.
- Finding: the rebase of Baseline after a parent merge in `rebaseAfterParentMerge` in `internal/landingshed/finalize.go` is pinned for a no-op merge and a moved head; check the case where the merge brings commits and the fixer then commits again.
