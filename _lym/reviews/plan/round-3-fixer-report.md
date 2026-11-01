# Fixer report — plan, round 3

## Changes

- **F1** (MEDIUM, `_lym/plan/06-front-task-package.md`, `TestRemove`): fixed differently.
  The row's fake `RemoveEntry` now asserts that `lock.TryAcquireWriteLock` on the record file's lock path reports the lock held, and starts `Settle` in a goroutine that the test waits for once `Remove` has returned.
  The row names the reason: the lock is a blocking flock (`internal/lock.AcquireWriteLock`, taken by `state.UpdateJSON`), so a `Settle` called inside it would never return.
  The goroutine `Settle` reads entry and record both gone, records nothing and runs no action.
  The second half of the row, a `Settle` after the removal, is unchanged.
  I did not make two parts of the proposal.
  First, the goroutine alone catches a missing lock only when the scheduler happens to run it between the drop and the removal, so the row also probes the lock with the non-blocking `TryAcquireWriteLock` (`internal/lock/lock.go:40`). The probe fails an unlocked removal on every run.
  Second, I left out the claim that the outcome holds "whichever way the goroutine interleaves" because `Settle` reads both present or both gone. The goroutine starts inside the held lock, so under a correct `Remove` it always reads both gone and never both present.
  This change came from the self-check finding of the fix-proposal lens.
- **F2** (LOW, `_lym/plan/11-adopt-holds.md`, `TestAdopt`): fixed as proposed.
  A new row covers a records branch of an adopted beam's pair that the reconcile left unmoved in state diverged, with `Worktree` the records sibling and `CodeWorktree` the pair's code worktree.
  It asserts that the warning names `lym fabric pull` in the `CodeWorktree` path and not in the `Worktree` path.
  The worktree fields match `originTargets` (`internal/fabricengine/originreconcile.go`).

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings; no finding left unchecked.
- fix-proposal, MEDIUM, F1's goroutine row catches a missing lock only some of the time, and its "both present" clause cannot occur: fixed with own change, the `TryAcquireWriteLock` probe in the fake `RemoveEntry` and the "both present" clause dropped (see F1 above).
- fix-proposal, F2: no finding.

## Disputed

None.

## Deferred

None.
