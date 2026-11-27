---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The owner source now comes ahead of step 1 as proposed, but a run read as owned elsewhere in step 1 is released and handed over instead of getting a held_back record. Its tip is already on origin, so nothing is left to push. A held_back record there makes TargetRefused read the landing target refused, and SettleRefused re-reads the same owner and writes it again on every tick, so every later landing on the prime stays owed until someone adopts the run back from its rightful owner (self-check, fix-proposal lens)."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Card 2 and card 3 follow the proposal, and so does card 20's Waits read. The new test rows go where the batten-run reads live, not into the engine tests. TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold gains rows for the batten record, and a new tier1 TestHubDobbyDeps_PushSummaryWaitsReadBattenAndPairRecords covers the wired Waits. TestRunWatch and TestPushSummary already pin a Waiting run owing no line and a Waits-listed run in the summary, and neither engine tells a batten run from a pair's, so a batten row there would duplicate an existing one."
  - id: F6
    disposition: fixed
---

# Round 3 fixer-report

## Changes

- **F1** (card 6):
  - The board's `Ahead` is `HasUncommitted` or `HasUnpushed`, `HasUncommitted` alone with pushes off by `ApplySkipEnv`, and false with git off. `starter.start` with `ifAhead` reads the same.
  - Created `plan:internal/boardengine#Board.PusherSync`. It commits as `Sync` does and pushes through `Bolt.Push` with `OwedUnrecorded` and `SingleAttempt`. `pusherDeps` uses it as the board's `Push`, with `ApplySkipEnv` applied.
  - Added `Bolt.HasUncommitted` and `ApplySkipEnv` under Uses.
  - Created the tier1 `TestPusherDeps_BoardPushCommitsTheBoardWrite`:
    - an uncommitted write is committed and pushed;
    - over an unreachable origin it is committed, stays owed and writes no record;
    - with pushes off it reads level.
  - Updated the ImpactSummary.
- **F1** (card 5, raised by the self-check): `SyncOptions` now gains `SingleAttempt`, which `pushClassified` reads, and `PairPush.SingleAttempt` copies into it. Without this, a Bolt push could not ask for one attempt. Added `internal/fabricengine#SyncOptions` under Edit.
- **F2** (cards 12 and 00):
  - Edits `MergeOptions` and `Fabric.Merge`, and creates `gitrepo#Repo.MergeStartNoFastForward` (`git merge --no-ff --no-commit`).
  - `MergeOptions.NoFastForward`, which `landUnderLock` sets on every parent-side merge, makes a landing without squash a merge commit carrying the trailer.
  - Uses gained the vocabulary test, `fabriclayout/doc.go` and `gitrepo/doc.go`.
  - `TestLandingDeps_LocalLanding` gained the squash-off row.
  - The overview's trailer decision says every local landing leaves one trailer commit.
  - Updated card 12's ImpactSummary.
- **F3** (card 11, fixed differently):
  - A new opening paragraph runs the Bolt pull or the beam reconcile in every pass and every `SettleRefused`, ahead of the re-record and every owner read and board-entry read. An unreachable origin ends the pass owed.
  - Step 3 reads from that state.
  - In step 1, a run owned elsewhere gets no `done` write. Its landing is removed, its records commit is held as step 2's drop holds it, and the run is handed over, with no refusal record.
  - A failed `RunOwner` read leaves the landing standing.
  - Added the `TestParentPush` row: the tip is already on origin and the takeover is on origin's board only.
- **F4** (card 11):
  - When step 1 removes the last `Dropped` run whose commit the tip carried, it clears the drop's refused-push record, and `SettleRefused` does the same.
  - `restartHubPushers` runs `SettleRefused` for pair targets holding `Dropped` too, so a beam parent whose only landing was dropped is reached (self-check).
  - Added rows to `TestParentPush` and `TestRestartHubPushers`.
- **F5** (cards 2, 3 and 20, fixed differently):
  - Card 2 edits `battenCLI.driverDepsFor`, which sets `WaitRecordPath` (`shedrun.DriveWaitRecord` over the prime and the slug) and `NetworkWaitCeiling`.
  - Card 3's `DobbyRuns` fills `Waiting` and `WaitDied` from the batten run's own record and driver lock, and the DobbyRuns integration test gains rows for it.
  - Card 20's `Waits` reads each batten run's record in the prime. The tier1 `TestHubDobbyDeps_PushSummaryWaitsReadBattenAndPairRecords` pins the wired read.
- **F6** (cards 9 and 17):
  - `RunPush.BackoffSleep` is copied into `SyncOptions.BackoffSleep`, nil in production, and card 9's ImpactSummary counts two run-push fields.
  - `catchUpPushSeams.BackoffSleep` goes on its `RunPush`. `HubCatchUp` leaves it nil, and the test passes one that returns at once.
  - The estimate is restated to about two seconds.

`lym loom validate-plan` reported `ok: true` after the first round of edits and again after the self-check edits.

## Self-check

The four lenses each returned a report.

- fix-claims, MEDIUM, `Bolt.Push` cannot ask for a single attempt (merged with fix-consistency, fix-affected-set and fix-proposal #3): fixed as proposed. Card 5 adds `SyncOptions.SingleAttempt`, and `PusherSync` sets it. The "once" overstatement is gone.
- fix-claims, LOW, the `BOARD_SKIP_PUSH` row sets process-global state (merged with fix-proposal #4): fixed as proposed. The row uses `t.Setenv`, names the global and runs serially.
- fix-affected-set, LOW, the re-record's board-entry read is not said to read the pulled state: fixed as proposed.
- fix-consistency, MEDIUM, step 1's guarded clear disagrees with step 2's unguarded one: fixed differently. Step 1 no longer writes a `held_back` record (fix-proposal #1), so the guard is gone and both clears are unconditional again.
- fix-consistency, LOW, card 12's ImpactSummary is stale: fixed as proposed.
- fix-consistency, LOW, card 6's ImpactSummary is stale: fixed as proposed.
- fix-proposal, BLOCKING, a step-1 `held_back` record for a run owned elsewhere whose tip is already on origin refuses the landing target for good: fixed differently. The landing is released and the run handed over, with no record (see F3).
- fix-proposal, BLOCKING, a beam target holding only `Dropped` never reaches `SettleRefused`: fixed as proposed. `restartHubPushers` covers targets holding `Dropped`, and rows were added.

## Disputed

None.

## Deferred

None.

## Rulings

None.
