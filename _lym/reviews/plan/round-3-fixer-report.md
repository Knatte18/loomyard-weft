# Round 3 fixer-report

## Changes

- **F1** (MEDIUM, card 3), fixed differently.
  `TakeOverRequest` gains `LockWait`, where zero means `StatusPushLockWait`.
  `TakeOverRun` takes the moved slug's teardown lock and then the parent slug's lock before step 1, holds both until it returns, and refuses with an error that wraps `ErrPairPushRunning` and names the run.
  Step 2 now archives both pairs through `archiveOwnerSides`.
  `ArchiveOwnerTips` leaves the Uses list, which gains `TeardownLockPath`, `ErrPairPushRunning` and `StatusPushLockWait`.
  A new row, `APairOrParentTeardownLockHeldElsewhereRefusesBeforeAnyEntryIsWritten`, covers both locks with a short `LockWait`.
  - **Proposal not made:** taking the parent's lock after step 2 returns.
    With the parent's lock held elsewhere, step 2's `ArchiveOwnerTips` waits out its own fixed 30-second wait (`internal/fabricengine/ownerarchive.go:75`) before the call's `LockWait` acquisition is reached.
    So the proposed parent-lock row could not reach the refusal quickly, and a refusal at that point would come after step 2 has already pushed archive tags.
  - **Why the own change is better:** taking both locks first and archiving through `archiveOwnerSides` makes the order explicit, puts every refusal before anything is written, and keeps one lock from being taken twice.
    `DropLocal` takes no teardown lock, so no later step re-enters a held lock.
- **F2** (LOW, card 3), fixed as proposed.
  Step 5 now says a conflicting rebase is aborted and the branch ends at its local commits that origin lacked, without the entry commit.
  Self-check: the guarantee is now scoped to "on that branch".
  The conflict row asserts that tip and that no local commit of the parent's records branch holds a takeover entry.
- **F3** (LOW, card 8), fixed as proposed, first alternative.
  `hubCheckDeps` leaves `Rerun` unset.
  The `Check` closure of `hubHoldCheckDeps` sets `Rerun`.
  `hubFixDeps` sets `FixHold` over `boardengine.FixHold` with a one-entry tries map.
  The `Fix` closure sets `Tries` and `Rerun` after `hubFixDeps` returns.
  The `hubHoldCheckDeps` fill paragraph says the same.
  A new tier 1 test, `TestHubHoldCheckDeps_FilledSeamsNameTheRerun` (added to Create), has two rows:
  `TheCheckOfASubjectOwnedElsewhereNamesTheRerun` and `TheFixAtTheLimitNamesTheRerun`.
- **F4** (LOW, card 8), fixed as proposed and narrowed by the self-check.
  The `Hold(first)` seam returns:
  - this machine's hold that covers `first`;
  - else this machine's first hold;
  - else the other machine's first hold.

  It carries a `ReadsAsHere` flag.
  Step 1 treats a foreign hold as `no_hold`, with the line naming that machine.
  Step 3's `HoldLost` line names the machine that read found.
  The `NoOwnHoldEndsQuietlyNamingTheAdopt` and `AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` rows assert that machine's name.
- **F5** (LOW, card 8), fixed as proposed.
  `greenDeps` drops `Dir`, and `hubGreenDeps` no longer sets it.
  `greenFixture` drops `Dir` and its `dir` and `otherDir` fields, which only the dropped count-reset assertion read.

## Self-check

- fix-consistency, LOW: step 1's lead still said "reads this machine's hold" after the seam was redefined. Fixed as proposed; it now reads "reads the `Hold` seam".
- fix-proposal, LOW: "first hold" picks the wrong hold when this machine has several standing (`internal/boardengine/holdwrite.go:24`). Fixed as proposed; the seam prefers this machine's hold covering `first`.
- fix-proposal, LOW: the F2 guarantee was claimed for the whole adopt, though the moved pair's records push may already have landed. Fixed as proposed; it is now scoped to "on that branch".
- fix-claims: no findings.
- fix-affected-set: no findings (F2 left unchecked, since it is not a set).

## Disputed

None.

## Deferred

None.

## Rulings

None.
