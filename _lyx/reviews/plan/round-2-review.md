---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/17-batten-cancel-core.md:50
    summary: >-
      A re-run cancel and lyx batten run refuse each other when a dead writer's cancel marker is left and the child has reached its parent, so the reached-parent way forward is a verb the marker refuses.
    fix: >-
      Ask the parent to record an Added after Discussion decision for this corner, then plan it.
      Proposed shape: in card 17's cancelRun, when step 3 refuses for work that reached the parent and ReadMarker reads a marker present with its writer gone, take that marker over with TakeMarker and release it with remove true before returning the refusal, so the refusal's way forward, lyx batten run to finish the landing, is no longer barred by card 15's guard.
      Nothing else is ended, removed or cleared, and the refusal names that the stale marker was removed.
      Add a TestCancelRun row asserting the marker is removed, no EndForCancel, RemovePair, RemoveRecords, ClearClaim or LiftPauses call is made, and the text still names lyx batten run.
      A live writer's marker stays untouched and refuses with the wait way forward.
      If the parent instead records that the marker stays, change the reached-parent refusal text in card 17 step 3 and a TestCancelRun row to name a verb that works from that state.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/18-batten-cancel-verb.md:55
    summary: >-
      The production batten-lock seam, which must release the driver lock before it reports the run lock busy so the phase retry never waits on this process's own flock, is inline in hubCancelDeps and no planned test reaches that branch.
    fix: >-
      In card 18, Create plan:internal/battencli#takeBattenLocks -> func takeBattenLocks(driverLockPath, runLockPath string, wait time.Duration) (release func(), busy string, err error) and plan:internal/battencli#TestTakeBattenLocks -> func TestTakeBattenLocks(t *testing.T).
      Change the hubCancelDeps paragraph so TakeBattenLocks is wired through takeBattenLocks over shedrun.DriverLock and shedrun.RunLock of the prime, each taken with lock.AcquireWriteLockWithin bounded by pairteardown.CancelLockWait.
      TestTakeBattenLocks is an untagged test over temporary lock paths with a zero wait: both free are held until release is called and free after it; the run lock held on another handle reports busy naming the run lock's path, holds nothing, and a try-acquire of the driver lock on a fresh handle then succeeds; the driver lock held on another handle reports busy naming the driver lock's path with the run lock never taken.
      This overlay passes lyx loom validate-plan --overlay with no finding the baseline lacks.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/06-shed-locked-check-and-precondition.md:14
    summary: >-
      The KindBusy constant's doc says the run lock is held by a live driver or another step, but cards 6 and 16 also report KindBusy for a cancel marker or a missing seed refused with no lock held, and the constant is on no card's target list.
    fix: >-
      Add the member glyph internal/shedverbs#KindBusy to card 6's Edit list, right after internal/shedverbs#Spec, and say in its Intent that the constant's doc now reads that busy means the run is held and step leaves it to its holder, which covers the run lock held by a live driver or another step, and a pre-drive check refusal because a cancel is ending the run or its seed is gone.
      The glyph resolves with lyx quarry resolve, and this overlay passes lyx loom validate-plan --overlay with no finding the baseline lacks.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/13-teardown-cancel-phase.md:49
    summary: >-
      The retry after a busy hook reruns the write-lock hold and the owned-process end but not the pair's driver-lock probe, while the decision's step 2 reruns the probe and the hook together, and TestEndForCancel pins the shorter sequence.
    fix: >-
      In card 13 step 2, make a non-empty busy run step 1 once more and then step 2 again as a whole, that is t.driverLockHeld(slug) with its t.driverLockBusyError refusal, then req.AfterEnd.
      In TestEndForCancel change the busy-once sequence to holdWriteLocks, endOwned, release, driverLock, afterEnd, holdWriteLocks, endOwned, release, driverLock, afterEnd, stopStrands, and so on, and add to the held-driver-lock row a case where the lock is free on the first probe and held on the second, refusing naming lyx batten cancel.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/04-pair-parts-and-branch-discard.md:3
    summary: >-
      Card 4 bundles two independent public surfaces of fabric, the read-only PairParts and TaskBranchOnOrigin reads and the destructive DiscardTaskBranch with its new destroy-gate kind, so the riskier half is not reviewable on its own.
    fix: >-
      Split with lyx loom insert-card, never by hand.
      Card 4 keeps PairParts, Topology.PairParts, Topology.TaskBranchOnOrigin, TestPairParts and the doc.go paragraph on the reads.
      The new card after it takes DiscardResult, Topology.DiscardTaskBranch, archiveTaskTip, deleteArchivedTaskBranch, branchDirtinessArchivedTip, dirtyArchivedTip, DeleteArchivedTaskBranchForTest, TestDiscardTaskBranch, TestDeleteArchivedTaskBranch, the Edit targets checkBranchDirtiness and checkRemoteBranchRequest, and the doc.go paragraph on the archive-and-delete, with the Uses it needs.
      Cards 15, 17 and 18 keep naming the same handles, so only their numbers shift, and the overview Card Index gains the line.
      This fix is not overlaid, since it goes through insert-card.
---

### [MEDIUM] A re-run cancel and `lyx batten run` refuse each other over a left marker

**Location:** `_lyx/plan/17-batten-cancel-core.md:50`, with `_lyx/plan/15-batten-start-guards.md:37` and `_lyx/plan/14-batten-cancel-marker.md:33`.

**Issue:** Card 15's guard refuses `run`, `step` and `goto` on any present marker, a gone writer's included, and card 14 words that refusal as `run "lyx batten cancel <slug>" from the prime to finish it`.
Card 17 step 3 reads work that reached the parent before step 4 reads or takes the marker, and refuses with `run "lyx batten run <slug>" to finish the landing`.
A marker left by a cancel that crashed at steps 4 to 5, or that stopped on survivors, end-only or a Windows re-list, over a pair whose child loop then finishes its parent-side merge, puts both refusals on one state: the cancel names `lyx batten run`, and `lyx batten run` names the cancel.
The batten driver's own steps refuse on that marker too, so the child can reach its parent while batten stays stuck, which makes the state reachable.
The only exit left is deleting the marker by hand, which `refusal-spec.md` does not count as a way forward.
The decision record's D2 and D3 give each half, so the plan carries a gap the record leaves open; it needs a recorded decision before a card can close it.
Round 1 raised the same loop and its fixer deferred it for that reason; card 17 step 3 is unchanged and no `Added after Discussion` entry records a decision, so the gap stands.

### [MEDIUM] The production batten-lock seam's busy path has no reaching test

**Location:** `_lyx/plan/18-batten-cancel-verb.md:55`, with `_lyx/plan/17-batten-cancel-core.md:76`.

**Issue:** Cards 13, 17 and 18 all state that a busy report holds no lock, so the first lock is released before a busy run lock is reported and the phase retry never waits on this process's own flock.
The branch that does it is production code in `hubCancelDeps`, and card 17's `TestCancelRun` row drives only a fake `TakeBattenLocks`, which cannot show the real closure releases the driver lock.
A closure that kept it would make every retry wait out `CancelLockWait` on its own handle and then refuse, and no planned test would fail.
Card 18's own tests cover pause entries, the claim clear, the tackler stop, the verb's early refusals and two integration paths, none of which reaches a busy lock.
The fix extracts the seam as a named helper in card 18 with its own test, which `lyx loom validate-plan --overlay` admits.

### [LOW] `KindBusy`'s doc goes stale and is on no card

**Location:** `_lyx/plan/06-shed-locked-check-and-precondition.md:14`, with `internal/shedverbs/step.go:28`.

**Issue:** `internal/shedverbs/step.go` documents `KindBusy` as the run lock held by a live driver or another `step` invocation.
Card 6 reports a `LockedCheck` refusal under batten's `StepBusyKind`, and card 16 maps the seeding closure's marker and missing-seed refusals to `KindBusy`, where the run lock may not be held at all.
Card 6 already edits `Spec`, whose `StepBusyKind` doc it rewords, and the `battenPreStep` comment is on card 16, but the constant's own doc is on neither.

### [LOW] The busy-hook retry skips the pair's driver-lock probe

**Location:** `_lyx/plan/13-teardown-cancel-phase.md:49`.

**Issue:** D4 step 2 is the pair's driver-lock probe and then the caller's hook, and on a busy hook the phase runs step 1 once more and then "this step again".
Card 13 reruns step 1 and only the hook, and `TestEndForCancel`'s busy-once sequence pins `holdWriteLocks, endOwned, release, afterEnd` with no `driverLock` call in the second round.
A holder of the pair's driver lock that the second `endOwned` does not reach, such as an unregistered process, is then not probed before the strand stop and the loop seizure.

### [LOW] Card 4 bundles the read surface and the destructive surface of fabric

**Location:** `_lyx/plan/04-pair-parts-and-branch-discard.md:3`.

**Issue:** `PairParts` and `Topology.TaskBranchOnOrigin` are local and network reads with one integration test, while `Topology.DiscardTaskBranch` is a destructive removal that adds two destroy-gate kinds and carries two more tests and a test-only export.
Each is a public surface of `fabricengine`, so the rubric's granularity rule gives each its own card.
The two share no private helper, and card 17 uses both through their handles, so the split moves no target.

## Focus verdicts

- Way-forward clauses: the cancel-marker refusals, the missing-seed refusal and the `LockedCheck` wrap each end in a trailing `way forward:` clause, and a planned test asserts each (`TestCancelMarker`, `TestPreDriveCheck` for `battenPreRun`, `battenPreStep` and the built shed's `Run`, and `TestStepCmd_BusyTreatment` for the step verb). Not a defect.
- Busy-retry and the batten-lock seam: the plan states the first lock is released before the busy report, in cards 13, 17 and 18. The seam has no reaching test (F2), and the retry omits the probe the record reruns (F4).
- Owner read with origin unreachable: card 17 step 2 reads the owner from the local Bolt after either unreachable signal and refuses when another machine owns the slug, and `TestCancelRun` has a row for each source. Not a defect.
- New flag and verb Intents: `cancelCmd` and `--remove` state what they let through and what bounds it (card 18), as do `LockedCheck` (card 6), `LeaveClaim` (card 12), the cancel phase (card 13), the destroy-gate kind (card 4) and the gate exemption (card 17). Not a defect.
- Re-running the cancel over a reached parent with a left marker: F1.
- Busy records write-lock refusal: card 13 step 1 names the lock's path, how to see the holder with `lyx tender status`, `ps` or Task Manager, and the cancel again, and its test row asserts those. Not a defect.
- Docs and comments a card changes: `RunLockedError.Error` and the boardengine package doc (card 10), the `StepBusyKind` field doc through `Spec` (card 6) and the `battenPreStep` comment (card 16) are on target lists; `KindBusy`'s doc is not (F3).
