# A leftover non-git directory at a pair's path no longer blocks the slug

`lym batten cancel --remove` could leave the task worktree's directory behind.
`git worktree remove` had deleted it, then a late writer refilled it: a build server writing `obj/` output, or a session's exit hooks writing `.lym/reed.lock`.
The directory was no longer a worktree, so `lym fabric prune` did not list it.
Worktree-Create then failed on it as an interrupted step, and `lym board rename` still saw a pair there.

## What changes

- **Remove deletes a late refill.**
  `Topology.Remove` already finishes a directory refilled right after git's clean exit.
  It now checks the task worktree's path once more after the sibling's teardown, and it removes a directory that came back since through the same contained, retrying `removeWorktreeLeftover`.
  The ownership proof is the same: this call removed the registered worktree at that path.
  A refill it cannot delete is logged and reported in `StrayPath`, and Remove still succeeds, since the pair is gone.
- **Cancel names a directory it left.**
  `batten cancel --remove` adds a warning that names a `StrayPath` the removal reports and says to delete it before the slug runs again.
- **Worktree-Create halts with a way forward.**
  Batten already reads a directory that does not resolve as a worktree as an absent pair, so Add refuses it with `ErrTaskPathTaken`.
  `createRefusal` now maps that refusal to a `PreconditionError` naming the path and the way forward: save anything worth keeping, delete it, and run `lym batten run <slug>` again.
  The run halts at the row instead of being handed back as a stuck step.
  The text of `ErrTaskPathTaken` now names `lym fabric prune` as the verb that reports such a directory.
- **Prune reports the orphan directory.**
  `Topology.Prune` has a third pass.
  It reports each hub directory that is named as a valid pair slug, is not hidden, and is not a registered worktree, marking it `Unowned` and naming in `Error` how to clear it.
  It never removes one in any mode, `--force` included, because nothing proves the directory is fabric's.
- **Rename sees only a real pair.**
  boardcli's `PairExists` seam now calls the new `fabricengine.PairWorktreeStands`.
  It is true only when the task worktree path exists and is a registered linked worktree, so a leftover directory no longer refuses a rename or a project change.

## Design decisions

- Worktree-Create does not clear the leftover directory itself.
  A directory that is not a worktree carries no ownership proof, and the fabric removal paths never delete one.
  The run halts with the path and the way forward instead.
- The stray-directory pass in prune only reports.
  An operator directory in the hub with a slug-like name shows up as an `Unowned` entry and is never touched.
- The cancel does not wait for writers it did not start, such as a build server that is not a registered process of the pair.
  The last check in Remove makes the race window smaller, and the cancel's warning covers a refill that comes later.

## Tests

- `TestRemove_DeletesTaskWorktreeDirRefilledAfterItsRemoval`: a refill after the teardown is deleted and recorded as a path removal, with no `StrayPath`.
- `TestPairWorktreeStands_ReadsOnlyARegisteredWorktree`: a registered pair reads as standing, while a plain directory at a pair's path and an absent path do not.
- `TestPrune_ReportsANonWorktreeDirectoryAtAPairPath`: dry run, apply and apply with force each report the directory as `Unowned` with the way to clear it, and none of them deletes it.
- A `TaskPathTaken` row in `TestCreateRefusal_LeftoverBranchRemedyNeverNamesCheckout`, and a `RemoveLeavingAStrayDirectoryWarnsOfIt` row in `TestCancelRun`.
- `TestAdd_LeftoverWorktreeDirErrorNamesRemedy` asserts the new `ErrTaskPathTaken` wording, and boardcli's `TestCLI/Projects` makes its pair with `git worktree add`, since a plain directory no longer counts as a pair.
