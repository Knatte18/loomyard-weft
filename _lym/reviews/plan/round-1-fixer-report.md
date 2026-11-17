# Round 1 fixer-report — plan

`lym loom validate-plan` reports `ok: true` after every change below.

## Changes

- **F1 (BLOCKING), fixed differently.**
  - Not made: a `holdStands` re-read after `HoldCheck` on every result.
    After a green check the re-read finds the hold that card 8 step 3's `ClearHold` just removed, so it would end every green adopt.
  - Made instead: the check's ends judge the hold themselves.
    - Card 2: `ClearHold` refuses another machine's hold with a `*HoldRefusal` naming it, and reports false only when no hold stands.
    - Card 8: `holdCheckResult` gains `HoldLost`. Step 3 handles a `ClearHold` refusal or false, and step 4 handles a `FixHold` refusal; either ends with no `BeamUp` and no `DropBoltHold`, the line naming the machine.
      The `Fix` seam and `failedEnd` return the refusal.
    - Card 9: a `HoldLost` result ends the adopt with no `runBeamSteps`, and `holdStands` re-reads once more after a failing check before `runBeamSteps`.
      The "no re-read follows" sentence is replaced.
  - Rows added: `AHoldTakenOverDuringAGreenCheckBringsNothingUp` and `AHoldTakenOverDuringAFailingCheckBringsNothingUp` (card 8), `AHoldTakenOverDuringTheCheckStartsNoDobby` (card 9).
- **F2 (BLOCKING), fixed differently.**
  - Not made: the `Kept`/`TookOver` test.
    A take-hold killed before the fix task's takeover leaves a re-run that reads `Kept` true, so the fix task's run stays stranded, against card 9's own "an adopt killed after step 1 still moves the runs it took".
  - Made instead: the filer is kept in tracked state.
    - Card 1: `Hold` gains `FixMachine`, the machine whose check filed `FixTask`. It sits on card 1, since card 1 uses `Hold` and the validator refuses a later card's edit of it.
    - Card 2: `FixHold` sets `FixMachine` to the writer. A take-hold rewrite fills an empty `FixMachine` with the former `Machine`.
    - Card 9: the `FixTask` exemption applies only when the filer (`FixMachine`, else `Machine`) reads as this machine.
      Otherwise the fix task's run is moved unless its origin record names this machine's takeover.
  - Rows added: card 9's `TakeHoldOfAnAdoptFixHoldMovesTheFixTasksRun` and `AReRunAfterAKillFollowingATakeHoldMovesTheFixTasksRun`. Card 2's take-hold and `FixHold` rows assert `FixMachine`.
- **F3 (BLOCKING), fixed as proposed for `parentWarning`.**
  - `internal/adoptcli#parentWarning` is in card 9's Delete list. TestAdopt drops `ParentTakenByARaceCarriesTheFinalizeWarning`, which asserts the retired `AdoptPaused` (`adopt_test.go:721-734`).
  - The `workerOutName` part's premise is false: card 9's Delete list already held it.
- **F4 (MEDIUM), fixed as proposed.**
  - Card 10 was removed with `lym loom remove-card 10`, and the old card 11 is now card 10.
  - Card 9 gains the six `internal/adoptrecord` deletes and card 10's Intent, plus the reason the removal lands in card 9.
  - The overview's first Shared Decision says the guard first runs at card 9's batch gate, and card 9's index line names the package.
- **F5 (MEDIUM), fixed as proposed, except one part.**
  - Card 2 makes `FixHold` and `ClearHold` re-read the local Bolt after the push, and adds the `holdSetterBeforePushHook` var, which is process-global, so its rows run serially.
  - Rows added: `FixHoldRejectedOverATakeHoldRefusesNamingTheOtherMachine` and `ClearHoldRejectedOverATakeHoldRefusesNamingTheOtherMachine`.
  - Not made: `ClearHold` reporting false. It refuses instead, so card 8 can name the machine, as F1 needs.
- **F6 (MEDIUM), fixed with an addition to the proposal.**
  - Made as proposed: card 3 step 2 archives the parent's pair (`root` for the prime), and step 1 fetches the parent's code branch too.
  - Added: step 5's retry acts only on the rejected branch, and rebases this machine's other commits origin lacks rather than resetting them away.
    A reset alone would drop owed transitions of other runs, which card 9's catch-up then never pushes. A conflicting rebase aborts and returns its error.
  - Row added: `AParentRecordsCommitOriginLacksIsArchivedAndKeptAcrossTheRetry`.
- **F7 (MEDIUM), fixed as proposed.**
  - Card 8's Edit list gains `fixBody` (now taking `rerun`), `fixRemovalError`, `removeFixTask` and `fronttask#actions`.
  - It also gains the tests those changes reach: `TestFixBody` and `fronttask#TestSettle` (`fronttask_test.go:439` asserts `lym adopt root`).
  - Card 8's Uses gains `fronttask/doc.go` and `plan:boardengine#HoldRefusal`.
  - The Intent names `Rerun` in those lines and in `runCheck`'s owner-read and owned-elsewhere lines. `greenDeps` gains `Rerun`, and the hand step names `lym beam status`.
  - Rows added: `TestGreenEnd` gains `ABoardReadFailureNamesTheRerun`, and `TestFailedEnd` gains `AFixHoldRefusalComesBackAsLost`.
- **F8 (LOW), fixed as proposed.** Card 9 rewrites `PATTERN-dobby-runs.md`'s opening summary line.
- **F9 (LOW), fixed as proposed for the Edit list; the wording differs.**
  - Card 2 edits `internal/boardengine/holdwrite.go` in place of `#WriteHold`.
  - The header now drops the only-writer claim and describes the file's setters, without listing the writers by name, which would be an enumerated-consumer list (self-check).
- **F10 (LOW), fixed as proposed.** Card 10 (formerly 11) edits `internal/runproc#EndSubject`'s doc.
- **F11 (LOW), fixed as proposed.** Card 8's `checkCmd` sentence and ImpactSummary say fronttask starts the check detached and reads neither its exit code nor its output, and the retry rationale is gone.
- **F12 (LOW), fixed as proposed.** Card 1 gains `AParentHoldOriginNoLongerHoldsIsLost`.
- **F13 (LOW), fixed as proposed.** Card 1 drops `parseReplayHolds`, uses `ParseOwners`, and `Apply` reads holds and registry through it.

## Self-check

- fix-claims, MEDIUM: the kept `parentWarning` row and its refused way forward. Fixed: `parentWarning` is deleted as F3 proposed, and its row goes (see F3).
- fix-claims, LOW: no `TestGreenEnd` row reaches the board-read line. Fixed: row `ABoardReadFailureNamesTheRerun` added.
- fix-consistency, MEDIUM: `Kept` cannot reach `movedRun`. Fixed with own change: the `FixMachine` rule reads only the hold, which `movedRun` already takes (see F2).
- fix-consistency, LOW: card 9 names a machine when the hold cleared. Fixed: the adopt ends with the result's line, which names the machine or says the hold cleared.
- fix-consistency, LOW: card 8 Uses lacks `HoldRefusal`. Fixed: added.
- fix-consistency, LOW: `failedEnd`'s refusal return is untested. Fixed: row `AFixHoldRefusalComesBackAsLost`.
- fix-consistency, LOW: card 3 step 1 does not fetch the parent's code branch. Fixed: step 1 fetches it.
- fix-proposal, MEDIUM: F6's reset drops owed transitions. Fixed with own change: rebase on retry (see F6).
- fix-proposal, MEDIUM: F2 strands the fix task after a kill. Fixed with own change: `FixMachine` (see F2).
- fix-proposal, LOW: F9's header enumerates writers. Fixed: the header describes the file's setters, and doc.go says only boardengine's hold setters write `holds`.
- fix-proposal, LOW: `lym beam status` is a lookup, not a step.
  Disputed, premise false: the action starts while the fix task lands under this machine's `adopt-fix` hold, and card 6's `ThisMachinesHoldNamesTheRerun` shows the re-run then.
  Once the hold has moved or cleared, no re-run on this machine is owed, and beam status shows the hold's machine.
  `fronttask.go:318` wraps the hand step as one command in "run `%s`", and the table knows only the subject (`fronttask.go:28`).
- fix-proposal, LOW: the hook is process-global. Fixed: card 2 says its rows run serially.
- fix-proposal, outside-lens note: four integration tests import `adoptrecord` outside the cards.
  Disputed, premise false: each enclosing test is a card target.
  - `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` and `TestOwnerArchiveBeforeRefusal_SkipsWhileAnAdoptCoversTheWorktree` are edited by card 5.
  - Both `TestUnderAdoptWatch_*` tests are deleted by card 5.
  - `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` is edited by card 6.
- fix-affected-set, MEDIUM: the kept `parentWarning` way forward is refused under the hold. Fixed: `parentWarning` is deleted (merged with fix-claims' MEDIUM).
- fix-affected-set, unchecked: the F3 full reachability search. It was not run.

Every fork returned a report.

## Disputed

- F3, `workerOutName` part, premise false: `internal/adoptcli#workerOutName` is already in card 9's Delete list, beside `workerArgs`.

## Deferred

None.

## Rulings

None.
