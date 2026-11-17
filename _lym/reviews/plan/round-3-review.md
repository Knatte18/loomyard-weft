---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/03-fabric-takeover-writer.md (Intent, the TakeOverRun lock paragraph, and the TestTakeOverRun rows)
    summary: "TakeOverRun takes two teardown locks with a refusal when one stays held, but no input or row reaches that refusal and the wait is fixed at 30 seconds"
    fix: "In card 3 add a LockWait field to TakeOverRequest (zero means StatusPushLockWait, as TakeOverRequest.Delays nil means the default backoff) and pass it to both lock acquisitions. Add a TestTakeOverRun row named APairOrParentTeardownLockHeldElsewhereRefusesBeforeAnyEntryIsWritten that holds the moved slug's lock through TeardownLockPath, then the parent slug's lock, with a short LockWait, and asserts an error wrapping ErrPairPushRunning that names the run and that origin and both local branches are unchanged. State in the Intent that the parent slug's lock is taken after step 2 returns, because step 2's ArchiveOwnerTips takes and releases that same lock itself, so taking it earlier would make step 2 wait out the lock on this call's own hold."
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/plan/03-fabric-takeover-writer.md (step 5 and the ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun row)
    summary: "Step 5 drops the entry commit before the rebase, yet the conflict row says the branch ends at its pre-retry tip, so the tip the test pins is ambiguous"
    fix: "In card 3 step 5 and the row, name the tip. State that a conflicting rebase is aborted and the branch ends at the parent branch's local commits that origin lacked, without this call's entry commit, which the drop already removed, so a later push from this machine never publishes a takeover entry for an adopt that did not finish. Make the row assert that tip and that no local commit of the parent's records branch holds a takeover entry."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/08-adopt-check-ends-on-hold.md (the hubHoldCheckDeps paragraph and the fixDeps and checkDeps field paragraphs)
    summary: "checkDeps.Rerun, fixDeps.Tries and fixDeps.Rerun are gained fields that no named function sets, and hubGreenDeps takes rerun while hubCheckDeps and hubFixDeps do not"
    fix: "In card 8 state where each gained field is set. Say that the Check closure of hubHoldCheckDeps sets checkDeps.Rerun from its rerun argument, and that the Fix closure sets fixDeps.Tries and fixDeps.Rerun from its tries and rerun arguments after hubFixDeps returns, while hubFixDeps itself sets FixHold over boardengine.FixHold with the subject's tries as a one-entry map. Alternatively give hubCheckDeps and hubFixDeps the parameters hubGreenDeps takes. Add one tier 1 row to TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry or beside it that drives the filled Check or Fix seam over a hubforge hub and asserts a line naming the re-run, since a missed assignment passes every fake-seam row."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/08-adopt-check-ends-on-hold.md (the holdCheckDeps seam list and runHoldCheck steps 1 and 3)
    summary: "The Hold seam is defined as this machine's hold found false when none, yet the HoldLost line and the takeover row need the other machine's name from the same read"
    fix: "In card 8 define the Hold seam as returning the first hold standing in the local Bolt, whichever machine wrote it, with a flag for whether it reads as this machine's through ReadsAsHere. Let runHoldCheck step 1 treat a foreign hold as no_hold with the line naming that machine, and step 3 set HoldLost with the line naming the machine read there. The AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding row then asserts that machine's name in the line."
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/08-adopt-check-ends-on-hold.md (the greenEnd and greenDeps paragraph and the TestGreenEnd row list)
    summary: "greenDeps.Dir loses its only use, the fix-try reset, so the field and its assignment in hubGreenDeps stay as dead code"
    fix: "In card 8 delete the Dir field from greenDeps and its assignment in hubGreenDeps, and drop it from greenFixture, instead of saying only that Dir's use for the count goes. greenDeps and greenFixture are already Edit targets of the card, so no target changes."
---

### [MEDIUM] TakeOverRun's teardown locks have an unreachable refusal and an unstated acquisition order

**Location:** `_lym/plan/03-fabric-takeover-writer.md`, Intent paragraph on `TakeOverRun`'s locks, and the `TestTakeOverRun` rows.

**Issue:** The card takes the moved slug's teardown lock exclusive "throughout", through `lock.AcquireWriteLockWithin` with `StatusPushLockWait`, and the parent slug's lock exclusive for steps 4 and 5.
The mode is right: `PATTERN-pair-remote` gives shared for a pair push outside a removal and else exclusive, and the card's rebase and drop mutate the parent's branch.
`HandOverTips` and `ArchiveOwnerTips` (`archiveOwnerPair`) refuse with `ErrPairPushRunning` when the wait runs out, so the same refusal exists here, but `StatusPushLockWait` is a 30-second constant and `TakeOverRequest` has no knob for it.
The topology's lock refusals are reached by tests that shorten `Topology.teardownWait` through `export_test.go`, while `HandOverTips`, which takes `StatusPushLockWait` directly, has no test of its refusal.
No `TestTakeOverRun` row holds either lock, so the refusal is unreached, and the only way to reach it as planned is a 30-second wait.
The card also never says when the parent's lock is taken.
Step 2 hands the parent pair to `ArchiveOwnerTips`, which takes that same lock and releases it.
Read as "the lock is held throughout", an implementation would take it at the start and then wait out the lock on its own hold in step 2.
No step takes a lock the call already holds as the card reads, because the moved pair goes through `archiveOwnerSides`, but the parent's order is stated only by implication.

### [LOW] The aborted-rebase row pins a tip the card leaves ambiguous

**Location:** `_lym/plan/03-fabric-takeover-writer.md`, step 5 and the `ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun` row.

**Issue:** Step 5 drops the entry commit, then rebases the other local commits onto the new upstream tip.
When that rebase conflicts it is aborted, which restores the state before the rebase: the branch without the entry commit.
The card says the branch is left "as it was", and the row asserts the "pre-retry tip", which reads as the tip with the entry commit.
The two differ by one commit, and the row's input (a local commit and an old-owner transition on the same status file, with the hook committing at attempt 1) reaches the conflict but does not decide which tip is asserted.
Leaving the entry commit local is the worse choice: a later push by this machine's dobby would publish a takeover entry for an adopt that stopped.

### [LOW] Gained deps fields have no named setter, and the wiring is covered by no row

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md`, the `hubHoldCheckDeps` paragraph.

**Issue:** The card gives `checkDeps` a `Rerun` field, and `fixDeps` `Tries`, `Rerun` and `FixHold`.
`hubGreenDeps` is given a `rerun` parameter and sets `greenDeps.Rerun` from it, and `FixHold` is set by `hubFixDeps`.
`hubCheckDeps(prime, subject, terminal)` and `hubFixDeps(prime, subject)` take neither a re-run nor a count, and the card names no function that assigns `checkDeps.Rerun`, `fixDeps.Tries` or `fixDeps.Rerun`.
The seam closures receive `rerun` and `tries` as arguments and could assign them, but the card does not say so.
Every `TestRunCheck`, `TestFailedEnd` and `TestRunHoldCheck` row builds its deps by hand, so a missed assignment in the hub wiring passes all of them and ships lines that name an empty re-run.

### [LOW] The Hold seam cannot supply the machine name the HoldLost line needs

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md`, the seam list and `runHoldCheck` steps 1 and 3.

**Issue:** The seam is "this machine's hold in the local Bolt, found false when none".
Step 3 sets `HoldLost` with "the line naming the other machine, or saying the hold cleared" when the re-read finds no hold of this machine, and the row `AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` has the seam read another machine's hold.
A seam that returns only this machine's hold cannot tell a foreign hold from a cleared one, so the line cannot name anyone.

### [LOW] greenDeps.Dir becomes dead

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md`, the `greenEnd` and `greenDeps` paragraph.

**Issue:** In `internal/adoptcli/green.go`, `greenDeps.Dir` is read only by `fixtries.Write(deps.Dir, 0)` in `greenEnd`.
The card removes that write ("drops `Dir`'s use for the count") and keeps the field, which `hubGreenDeps` still sets.
A field nothing reads should go with its last use.

## Rejected

- Guard timing (focus 1): not a defect.
  `internal/gatecli/doc.go` plans a guard run per tier holding `//lym:guard` tests after any `--card` scope, so the test-only-code guard runs at every card gate.
  Cards 2, 3, 5 and 7 each list their unreachable functions with a Why naming the card that reaches them, as the `internal/runfsm.Order` entry does.
  Card 5's four `adoptrecord` entries are exactly the functions its deletion of `UnderAdoptWatch` orphans: `Record.TakesUp` and `ownerSubject` stay reached through `here.go` until card 9.
  Card 8 drops `FixHold`, `ClearHold` and `HoldRefusal.Error`, which `hubHoldCheckDeps` under `checkCmd` reaches, and card 9 drops the rest with the package.
  The adoptcli helpers that card 9's deletions would orphan (`checkSubjects`, `projectOf`, `refuseStandingMove`, `collect`, `deployStale`) are in its `Uses` list, and `foreignParent` and `parentOwnerOf` stay reached through `heldBeamWayForward`.
- `FixHold` and `ClearHold` (focus 2): not a defect.
  Card 2 reports success only when the re-read matches what the setter wrote, and its two `...WhoseReplayLost...` rows reach both cases where origin's replay dropped the change: no hold standing, and this machine's own changed hold standing.
  Card 1's replay reports a parent hold origin no longer holds as `Lost`, which is what makes those two inputs real.
- `runHoldCheck` order (focus 3): not a defect.
  Step 3 pulls and re-reads the hold before `Green` removes the fix task, and `AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` reaches that order.
  The window after the re-read is left to `ClearHold`, as the card says.
- `runForeground` step 2 and `movedRun` (focus 4): not a defect.
  A takeover entry naming another machine moves the fix task's run before the filer clause applies, and `TakeHoldBackFromTheMachineThatTookTheFixTasksRunMovesItAgain` reaches that input with `AddPair` uncalled and `TakeOver` called.
- Lock reentrancy in `TakeOverRun` (focus 5): the moved pair's archive goes through `archiveOwnerSides`, so it takes no lock the call holds; the parent's order is the F1 finding.
- Card 9 against `adoptrecord` importers (focus 7): not a defect.
  Every importing file in beamcli, orchcli and adoptcli, the external `adopt_integration_test.go` and `export_test.go` included, has its importing function in an Edit or Delete list of card 5, 6, 8 or 9.
