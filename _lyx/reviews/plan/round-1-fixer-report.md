# Fixer report — plan, round 1

## Changes

- **F1** (BLOCKING): fixed as proposed for cards 6 and 16 and for card 14's gone-writer text; fixed differently for card 14's live-writer text.
  - Card 6's Intent now says a `LockedCheck` error ends in its own `way forward:` clause, which the `ErrLockedCheckRefused` wrap keeps trailing.
    Its new `TestStepCmd_BusyTreatment` row asserts that the reported message ends with that clause.
  - In card 16 the seed-gone text now reads `…; way forward: run "lyx batten run <slug>" for a fresh start`, and every `TestPreDriveCheck` refusal row asserts the trailing clause.
  - Both `cancelMarkerRefusal` texts in card 14 now end in `way forward:`, and `TestCancelMarker` asserts the clause.
  - **Fixed differently:** the review proposed that the live-writer text tell the caller to check the holder with `lyx batten status <slug>`.
    No card has batten status report a cancel marker or its pid, so that command would not show the holder.
    The text instead names the pid and says to see it with `ps -p <pid>`, or in Task Manager on Windows (the marker records the pid on every platform), then run the verb again.
- **F2** (MEDIUM): fixed as proposed.
  - Card 13's `CancelRequest` now says a hook that reports busy holds nothing.
  - Card 17's `TakeBattenLocks` doc and card 18's wiring of it now release the driver lock already taken when the run lock is busy.
  - A new `TestCancelRun` row has `TakeBattenLocks` report busy once and then succeed, and asserts that the second call holds no lock from the first.
- **F3** (MEDIUM): fixed as proposed.
  - Card 17's step 2 now reads `Owner` from the local Bolt when origin is unreachable from either source, and refuses with the owned-elsewhere wording, with nothing ended, when the owner is another machine.
  - New `TestCancelRun` rows cover both unreachable sources with the local owner elsewhere, and assert the refusal and no `EndForCancel` call.
- **F4** (MEDIUM): fixed as proposed.
  - Card 18's Intent now states what `--remove` lets through, and what bounds it: the snapshot first, the archive tags before any delete, and D2's reached-parent refusal.
  - Each bound was checked against D5 and against cards 4 and 12.
- **F6** (MEDIUM): fixed, with the wording revised after the self-check (see Self-check).
  - Card 13 step 1's busy write-lock refusal names the lock path and says how to see the holder: `lyx tender status` or `ps`, or Task Manager on Windows.
  - It says to rerun the cancel once the lock is free, and to end the holder only when the same process keeps the lock across re-runs.
  - The `TestEndForCancel` row now asserts these names.
- **F7** (LOW): fixed as proposed.
  `internal/boardengine/doc.go` is on card 10's Edit list, and its Intent rewords the sentence about the ways forward to name `lyx batten cancel <slug>`.
- **F8** (LOW): fixed as proposed.
  `internal/shedverbs#Spec` is on card 6's Edit list, and the `StepBusyKind` field doc now names both sentinels.
- **F9** (LOW): fixed as proposed.
  `internal/battencli#battenCLI.battenPreStep` is on card 16's Edit list, and its doc comment's kind mapping now has a refused seeding report `KindBusy`.

## Self-check

- fix-consistency, LOW: card 13 step 2's still-busy hook refusal, the twin of F6, said nothing about how to see the holder.
  Fixed as proposed: it now names `lyx tender status`, `ps` and Task Manager, and the `TestEndForCancel` row asserts them.
- fix-proposal, MEDIUM: F6's proposed wording, ending the holder, is wrong for the prime's records write lock, which the whole hub shares, so the holder can be another run's healthy commit.
  Fixed as proposed: the refusal says to see the holder, rerun the cancel once it lets go, and end the holder only when the same process keeps the lock across re-runs.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **F5** (MEDIUM, decision): deferred because it needs a decision from the parent.
  - The finding asks the parent to record an Added-after-Discussion decision: before the reached-parent refusal, take over and remove a gone writer's cancel marker.
  - That changes D2's rule that refusals change nothing.
  - The decision record (`_lyx/discussion/decision-record.md`) is outside this fixer's write surface.
  - This round's prompt also forbids sending any message about the review to another session, so the decision cannot be requested from here.
  - Card 17's step 3 is left as it is until the parent records the decision.
