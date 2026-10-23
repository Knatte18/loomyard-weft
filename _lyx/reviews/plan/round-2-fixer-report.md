# Fixer report — plan, round 2

Card numbers below are the new numbers, after F5's split: the old card 4 became cards 4 and 5, and every later card moved up by one.

## Changes

- **F2** (MEDIUM): fixed differently.
  - Card 19 (was 18) creates `takeBattenLocks(driverLockPath, runLockPath string, wait time.Duration) (release func(), busy string, err error)` and `TestTakeBattenLocks`.
  - `hubCancelDeps` now wires `TakeBattenLocks` through `takeBattenLocks(shedrun.DriverLock(location, slug), shedrun.RunLock(location, slug), pairteardown.CancelLockWait)`.
    The helper takes the driver lock first, and when the run lock is busy it releases the driver lock before reporting.
  - `TestTakeBattenLocks` has three cases:
    - Both locks free: both are held until `release` is called, and free after it.
    - The run lock held on another handle: busy naming the run lock's path, and the driver lock is then free on a fresh handle.
    - The driver lock held on another handle: busy naming the driver lock's path, and the run lock is free, since it was never taken.
  - **Fixed differently:** the review proposed a zero wait for this test.
    A zero wait never tries the lock.
    `lock.AcquireWriteLockWithin` (`internal/lock/lock.go:51`) builds a `context.WithTimeout` of zero, and gofrs/flock v0.8.1's `tryCtx` returns `ctx.Err()` before its first `TryLock`.
    That error reads as busy, so the "both free are held" case could never pass.
    The test therefore uses a wait of a few milliseconds and states why.
- **F3** (LOW): fixed as proposed, with the wording revised after the self-check.
  - `internal/shedverbs#KindBusy` is on card 7's (was 6) Edit list, after `Spec`.
    The glyph resolves with `lyx quarry resolve`.
  - The Intent says the constant's doc now reads that busy means the run is held and step leaves it to its holder.
    That covers the run lock held by a live driver or another `step` invocation.
    It also covers a check the spec runs before the drive, under the run lock or in its seeding, refusing because the run is held elsewhere or no longer seeded.
- **F4** (LOW): fixed as proposed.
  - Card 14 (was 13), step 2: a busy hook now runs step 1 once more, then step 2 again as a whole, that is the driver-lock probe with its refusal, then the hook.
  - The busy-once sequence in `TestEndForCancel` now reads `holdWriteLocks, endOwned, release, driverLock, afterEnd, holdWriteLocks, endOwned, release, driverLock, afterEnd, stopStrands, …`.
  - The held-driver-lock row now also has the lock free at the first probe and held at the second, after a busy hook.
  - So that the row can be scripted, `fakeSubstrates` gains a `driverLockHeldByProbe` list.
    The list answers each probe in turn, ahead of the existing `driverLockHeld` flag, so existing rows keep their calls.
- **F5** (LOW): fixed as proposed, through `lyx loom insert-card` and `lyx loom remove-card`.
  - The split ran in three steps:
    - `insert-card 4 pair-parts-reads` created the new card 4.
    - `insert-card 5 task-branch-discard` created the new card 5.
    - The old combined card, by then card 6, was removed with `remove-card 6`.
    This gives both halves file names that match their content.
  - Card 4 holds `PairParts`, `Topology.PairParts`, `Topology.TaskBranchOnOrigin`, `TestPairParts` and the `doc.go` paragraph on the reads.
  - Card 5 holds `DiscardResult`, `Topology.DiscardTaskBranch`, `archiveTaskTip`, `deleteArchivedTaskBranch`, `branchDirtinessArchivedTip`, `dirtyArchivedTip`, `DeleteArchivedTaskBranchForTest`, `TestDiscardTaskBranch` and `TestDeleteArchivedTaskBranch`.
    It also has the Edit targets `checkBranchDirtiness` and `checkRemoteBranchRequest`, the `doc.go` paragraph on the archive-and-delete, and the Uses it needs.
  - The tools updated the Card Index.
    Every prose mention of a moved card number was updated by hand: in the overview's shared decisions and in cards 9, 10, 12, 13, 14, 15, 16, 17 and 18.
    `lens-review.md` is left as it is, since it records an earlier review under the old numbers.
- `lyx loom validate-plan` passes after every change.

## Self-check

- fix-consistency: no findings.
- fix-claims, MEDIUM: the zero-wait fact that F2's fix states also breaks two tests this round had not changed.
  One is `TestHoldRecordsWriteLock` in card 6 (was 5), whose "held once the first is released" can never pass with a zero wait.
  The other is `TestHoldRecordsWriteLocks` in card 14 (`writeLockWait` zero, so no lock is ever taken).
  Fixed as proposed: both tests now use a wait of a few milliseconds and state why.
- fix-proposal, LOW: F3's proposed doc wording named batten's reasons, a cancel ending the run and a gone seed, in the doc of a shared `shedverbs` constant.
  Loom's spec also uses `KindBusy` (`internal/loomcli/arm.go:298`).
  Fixed as proposed: the wording is now neutral, as given under F3 above.
- fix-affected-set: no findings; F1 was left unchecked, since it was deferred.

## Disputed

None.

## Deferred

- **F1** (MEDIUM, decision): deferred because it needs a decision from the parent, for the same reason as round 1's F5.
  - The finding asks the parent to record an Added-after-Discussion decision on the reached-parent corner with a dead writer's marker left.
    The proposed decision is that the cancel takes over and removes that marker before refusing; the alternative is to keep the marker and change the reached-parent way forward.
  - Either choice changes D2's rule that refusals change nothing, or the way forward that D2 and D3 give.
  - The decision record (`_lyx/discussion/decision-record.md`) is outside this fixer's write surface.
  - This round's prompt forbids sending any message about the review to another session, so the decision cannot be requested from here.
  - Card 18's (was 17) step 3 is left as it is until the parent records the decision.
