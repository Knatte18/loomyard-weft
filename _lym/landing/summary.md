# Batten reads a stray dir as a gone pair; a beam landing keeps its mark

Closes #21

Issue #21 reported a beam landing that stuck at Worktree-Teardown with no `lym` way forward.
This change fixes the batten half; the `fabric remove` half is task `fabric-remove-ignored-files`.

## What changes

- A task worktree path that exists but does not resolve as a worktree, such as a directory `lym fabric remove` left holding ignored build output, now reads as a gone pair.
  `taskWorktreePresent` returns `false` for it instead of the resolver's error, so `lym batten run`, `cancel` and `cleanup` no longer fail with "not a git repository".
  The re-adoption's gone check reads through the same function, which replaces the separate `taskWorktreeGone`.
  The refusal that names an absent pair now also covers a stray directory.
- The start guard no longer reads the beam mark as a promote leftover in two cases.
  A slug the board no longer holds as a task passes, such as a beam project whose own Land pruned its entry.
  A run whose work already reached its parent passes too, judged as `lym batten cancel` judges it.
  A beam's own landing run therefore resumes Worktree-Teardown over the mark its Beam-Up wrote.
- This breaks the loop where the promote refusal named `lym batten cancel --remove`, and cancel refused with "run `lym batten run`".
  The refusal still names that cancel for a real leftover, where the work has not reached the parent, and cancel accepts it there.

## Design notes

- `hubChildReachedParent` is extracted from `hubCancelDeps` so cancel and the start guard share one reading of "reached its parent".
- `startGuard` gains a `ReachedParent` seam; a nil seam reads every run as not reached.
- `Promoted` now reads `false` unless the entry is a task, where it read `false` only for a project entry.
- The overview and the beam-land member listing still only look for the directory, and do not read a stray one as gone.
- `taskWorktreeLocator` no longer calls `presentTaskWorktreePath` on later calls; it only stats, so it spawns no git after the first success.

## To check by hand

- Not run: the stuck case end to end on a real hub (a landed beam, the worktree directory left as a non-git directory with ignored files, then `lym batten run <slug>`).
- Unit and integration tests cover the start guard and the re-adoption through the stray path.
