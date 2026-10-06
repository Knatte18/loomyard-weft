# Round 2 fixer report

Every finding in `round-2-review.md` is fixed; the edits are to the plan's card files and `00-overview.md` only.
`approved:` in `00-overview.md` is untouched.

## Changed

- F1 (lease-lost discriminator): card 9 now declares `plan:internal/gitrepo#ErrLeaseRejected`, returned bare by `UpdateRemoteBranchLeased` for a failed lease and never for another failure, with `push_test.go` rows for both.
  Card 11 uses it: `ErrLeaseRejected` maps to the divergence refusal, any other update error and a failed fetch to the re-run-once-reachable refusal, and a new row (a `pre-receive` hook that exits non-zero) reaches the non-lease update failure.
- F2 (boundary test): card 9 edits `cmd/lyx/gitrepoboundary_test.go` so both new method names join `gitrepoPinnedRunBoundMethods`; `00-overview.md`'s integration verify line now runs `./cmd/lyx/...`.
- F3 (destructive chokepoint): card 11 makes the remote update a new gated executor `updateRemoteBranch` in `destroy.go` (leading `rec`, gate run before the push), orders `ResetPairWarp` as gate, fetch and ancestry read, executor, `resetHardTo`, and edits `cmd/lyx/destructiveguard_test.go` for the banned token and executor row; the pinned executor tally in `destroy.go`'s comment is reworded.
- F4 (backward-Uses run tests): card 1 now edits `internal/websterengine/runlevel_test.go` and deletes `TestRun_ReorderingIsObservableInMasterPrompt` and `TestRun_CycleReportingSurfacesOnRunResultButNeverFails`, naming `sequence_test.go` as the remaining coverage.
- F5 (paused): card 8 names paused as an outcome, in the strand-removal sentence, the outcome list, the warning rule and the `triage_test.go` table.
- F6 (rebaseline done row): card 6 drops the `done` row from its new table and extends `TestRebaseline_RefusesChangedBegunCardBody` with the new done-batch reason, keeping its existing substring.
- F7 (reset test rows): card 11 extends `TestResetPairWarp_DiscardsCommitsAndOwnPathDirtKeepsUntracked` and `TestResetPairWarp_DirtyPathOutsideOwnPathsRefuses` instead of adding rows for the same behavior.
- F8 (whole-plan Delete row): card 3 extends `TestValidateCmd_ScopeFollowsRunProgress` instead of adding a row to the new plan-gate table.
- F9 (spec rows): card 2 adds a `delete-before-reference` row to the end of the Validation checks list, and card 3 adds `delete-target-gone` after it, each noting `internal/planglyph` emits it; the paragraph is dropped.

## Deferred

Nothing deferred.

## Notes

The plan-review rubric stencil still pins the check tally ("twenty-eight", "twenty-seven"), which card 1's new check makes stale.
No card edits it: round 1 removed that edit because no Decision or Constraint licenses it, and the rubric binds over this round's fixes, so the finding stays with the board note that owns the plan-spec and template prose.
