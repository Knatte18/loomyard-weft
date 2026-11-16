# Fixer-report, round 1

## Changes

All three findings fixed as proposed. Gate `lym gate test ./internal/beamcli` passed (tier0, 0 failed).

- **F1** (commit `9737a885`): `prOpenCmd`'s `Long` text in `internal/beamcli/cli.go` lists two more refusals, a merge task that is not done and a conflict with the parent branch.
  A closing sentence says both are checked before a push or a reopen and that a beam only behind its parent still opens.
- **F2** (commit `2283229b`, shared with F3 since both edit the same two test files): two `TestPROpen` rows.
  `OpenMergeTaskRefusesBeforeTheConflict` sets an open `b-merge-in` task and a conflict, wants `lym batten status b-merge-in` and not `lym beam merge-parent`.
  `BeamOnlyBehindItsParentStillOpens` sets `parentUnmerged` and wants the pull request created and recorded.
  The table gained a `wantNoErr` field, checked for every refusal row.
- **F3** (commit `2283229b`): `landWorld` gained `tasksErr` and `parentConflictsErr`, returned by the `Tasks` and `ParentConflicts` fakes.
  Two rows use them: `FailingParentConflictReadRefusedNamingTheRerun` as the review proposed, and `FailingBoardReadRefusedNamingTheRerun` so the new `tasksErr` field has a covering row too. Both set `wantUntouched`.

- **Gate repair** (after the verify gate): `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` in `internal/reedengine/bindings_integration_test.go` failed once with "the recorder never finished writing: condition never became true within 5s".
  The test is unrelated to this round's diff, passes alone under `lym gate test ./internal/reedengine --tier 2`, and ran 13.5s under the parallel tmux load of the full tier2 run.
  The deadline of every `waitUntil` for the recorder script in that file is now 30 seconds, since the 5-second wait lost to that load.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
