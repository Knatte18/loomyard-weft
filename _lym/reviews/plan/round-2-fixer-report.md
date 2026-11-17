# Fixer report — plan, round 2

Every finding's premise held against the code; each is fixed as proposed, F1 with one member added.
`lym loom validate-plan` passes after every change.

## Changes

- **F1 (BLOCKING), fixed as proposed, one member added.**
  - The overview's first Shared Decision now says:
    - the go scoper runs every `//lym:guard` test after any `--card` scope, so the guard runs at each card and batch gate;
    - a package no main imports is outside the analysis;
    - each card that leaves a function with no reached caller lists it in `testOnlyCodeAllowed`, following `internal/runfsm.Order`.
  - Card 9's Intent drops the false reason for removing `adoptrecord` there.
  - Each of cards 2, 3, 5, 7, 8 and 9 gains the Edit target `cmd/lym#testOnlyCodeAllowed`, and the entries are added or removed as the review lists them.
  - Card 5 also lists `internal/adoptrecord.Record.Covers`.
    Its only caller is `Watch.covers` (`internal/adoptrecord/watch.go:65`), so it becomes unreachable with the watch.
  - Every card says the exact keys come from the guard's report at its own tree.
- **F2 (MEDIUM), fixed as proposed.**
  - Card 8 step 3 now pulls Bolt and reads the `Hold` seam again before the first `Green` call.
  - When no hold of this machine stands, the result ends `HoldLost` and calls neither `Green`, `ClearHold`, `BeamUp` nor `DropBoltHold`.
  - The row `AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` is added.
- **F3 (MEDIUM), fixed as proposed.**
  - Card 2's `FixHold` and `ClearHold` now compare the re-read with what they wrote, and refuse on any other read.
  - The intent states why the push gives no sign of the lost change: the replay reports it as `Lost` and only logs refused findings.
  - The rows `FixHoldWhoseReplayLostItsHoldRefusesInsteadOfReportingSuccess` and `ClearHoldWhoseReplayLostItsRemovalRefuses` are added.
- **F4 (MEDIUM), fixed as proposed.**
  - Card 9's fix-task clause in `movedRun` gains the qualifier "and its origin record holds no takeover entry naming another machine".
  - The row `TakeHoldBackFromTheMachineThatTookTheFixTasksRunMovesItAgain` is added.
- **F5 (MEDIUM), fixed as proposed.**
  - The seam becomes `Green(subject, rerun)`.
  - `internal/adoptcli#hubGreenDeps` -> `func hubGreenDeps(prime *lymcwd.Location, subject, rerun string) (greenDeps, error)` is added to card 8's Edit list.
  - The intent says it sets `greenDeps.Rerun`, and `hubHoldCheckDeps` calls it with the re-run.
- **F6 (MEDIUM), fixed as proposed.**
  - Card 9's Edit list gains `internal/adoptcli_test#TestCommand_HereAdoptsTheSubjectOfItsWorktree`.
  - Its intent: assert that the hub's Bolt holds no hold after `--here`, in place of the `adoptrecord.Read` assertion.
- **F7 (MEDIUM), fixed as proposed.**
  - Card 3 states that `TakeOverRun` holds the slug's teardown lock exclusive through `lock.AcquireWriteLockWithin` with `StatusPushLockWait`.
  - The moved pair is archived through `archiveOwnerSides` and the parent's pair through `ArchiveOwnerTips`.
  - The parent's records-branch commit, push and retry hold the parent slug's teardown lock.
  - `internal/fabricengine#archiveOwnerSides` is added to Uses.
- **F8 (MEDIUM), fixed as proposed.**
  - The row `ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun` is added to `TestTakeOverRun`.
- **F9 (LOW), fixed as proposed.**
  - Card 2 moves `internal/boardengine/doc.go` from Extends to Edit and drops the empty Extends group.
  - The intent names the rewritten "WriteHold is the one writer of the holds" sentence.
  - It also says the `# Machine registry and holds` field list gains `fix_machine`.

## Self-check

- fix-consistency, MEDIUM: the card 3 conflict row asserted that origin holds no entry, but the pair's records-branch push can land.
  Fixed as proposed: the row now asserts on the local and origin parent records branch.
- fix-consistency, LOW: card 8 step 3's new pull had no failure outcome.
  Fixed as proposed: a failing pull ends not green with its error and the re-run, and calls no later seam.
  Row `APullBeforeTheGreenEndThatFailsCallsNoLaterSeam` added.
- fix-consistency, LOW: card 9 dropped "every entry" cards 2, 3, 5 and 7 added, though card 8 already drops three.
  The overview credited the removal only to "the card that reaches it".
  Fixed as proposed: card 9 now says "every entry still standing", and the overview says "the card that reaches it, or removes its package".
  The same point came from fix-proposal (LOW) and fix-affected-set (LOW), merged here.
- fix-claims, LOW: card 3 cites `PATTERN-pair-remote` for an exclusive lock on the parent's push, but the pattern takes such a push shared.
  Fixed as proposed: the card now gives the reason for exclusive, which is that step 5's retry drops and rebases local commits of that branch.
- fix-proposal, MEDIUM: F3's own-hold-changed refusal was read as `HoldLost` in card 8, with a line naming another machine or a cleared hold.
  Fixed with an own change.
  - Card 2's `HoldRefusal` now carries `Machine` and `OwnStands`.
  - In card 8, only a refusal with `OwnStands` false is `HoldLost`.
    `ClearHold` with `OwnStands` true ends like a failed clear, naming the re-run.
    `FixHold` with `OwnStands` true sets no `HoldLost` and still brings the subjects up.
  - Row `ARefusalOverThisMachinesChangedHoldIsNotLost` added, and card 2's two new rows assert `OwnStands`.
- fix-proposal, MEDIUM: the F4 row's hold named the other machine, so `holdStands` would stop before `movedRun`.
  Fixed as proposed: the row's hold after `WriteAdopt` names this machine (a take-hold back), `FixMachine` this machine.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
