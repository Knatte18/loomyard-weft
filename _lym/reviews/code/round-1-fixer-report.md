# Fixer-report, round 1

## Changes

- **F1** (fixed as proposed): `TestRemove_KeepsAndReportsTaskWorktreeDirRefilledAfterItsRemovalWhenItCannotBeDeleted` in `internal/fabricengine/remove_leftoverdir_integration_test.go` refills the directory in the teardown hook and fails the removal through the leftover seam with `fs.ErrPermission`.
  It asserts a nil error, `StrayPath` equal to the task worktree path, the directory still on disk and no removal in the mutation record.
  The refill body shared with the existing refill test moved into `refillWithLockFile`.
- **F2** (fixed as proposed): `TestPrune_IgnoresDirectoriesThatAreNoPairPath` in `internal/fabricengine/prune_unowned_integration_test.go` creates a live pair, `.hidden`, `notes-weft` and `_portals`, and asserts Prune reports no entry whose `CodeWorktree` is any of them.
- **F3** (fixed as proposed): `PairWorktreeStands` in `internal/fabricengine/remove.go` lists the repository's worktrees itself and reads true for a present path when the listing fails; its doc comment says so.
  No test added: the finding is a design finding, not a coverage gap.
- **F4** (fixed differently): the reviewer proposed returning the original error when `PairWorktreeStands` is true.
  I did not make that change: it leaves the halt with no way forward, which the self-check's fix-proposal lens showed, and the review's own alternative was to stop claiming the path is no worktree.
  `createRefusal` in `internal/battencli/wire.go` now takes `taskPathStands func() bool`, asked only in the `ErrTaskPathTaken` branch.
  A registered worktree there gets a precondition error whose way forward is `incompletePairRemedy` plus a resume; a plain directory keeps the delete-it text.
  `TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout` has the `TaskPathTakenByARegisteredWorktree` row.
- **F5** (fixed as proposed): `stepProjects` in `internal/boardcli/verbs_test.go` creates a plain directory at a new entry's pair path and expects its project change and its rename to succeed.

Commits, one per finding, plus a follow-up commit for F4: `git log` on the branch.

Gate: `lym gate test --tier 0 --tier 1` over `fabricengine`, `battencli` and `boardcli` passed before the self-check; `--tier 0` over `battencli` passed after the F4 change.

## Self-check

- fix-code-correctness, LOW: the wiring of `createRefusal`'s flag at its caller is not pinned by a test. Not a finding under the rubric: it asks for an improvement the review never made worse.
- fix-code-correctness, LOW: the unlistable-repository branch of `PairWorktreeStands` has no test. Not a finding under the rubric, for the same reason; F3 is not a coverage gap.
- fix-consistency: no findings. It noted that `readoptstop.go` words `ErrTaskPathTaken` as "delete it" without checking for a registered worktree; that twin lies outside F4's location and was left as it was.
- fix-proposal, LOW: F4's pass-through carries no way forward. Fixed with my own change, as under F4 above.
- fix-proposal, LOW: `PairWorktreeStands` ran on every create error. Fixed with my own change: the argument is now a function asked only for `ErrTaskPathTaken`.
- fix-proposal, LOW: F3 with F4 passed a raw error for an unlistable repository holding a plain directory. Resolved by the F4 change: that case now gets the remove-the-pair remedy.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
