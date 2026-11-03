# Batten reads a stray directory as gone; a beam lands past its own mark

A landed beam's run no longer stalls at Worktree-Teardown.
Batten verbs no longer fail on a task worktree path that is now a plain directory.
This is the batten half of issue #21; the `lym fabric remove` half, which left the directory behind, is task `fabric-remove-ignored-files`.

## What changes

- **A stray directory is a gone pair.**
  `taskWorktreePresent` in `internal/battencli/wire.go` now reads a path that stands but does not resolve as a worktree as absent, as it reads a missing path.
  Before, it returned the resolver's "not a git repository" error.
  That error failed a resume at Worktree-Teardown, `lym batten cancel` and the dobby's child reads.
  A teardown resume over such a directory now judges the owner on the local board and goes on, as it does when nothing stands at the path.
  The landed `taskWorktreeGone` already read a stray path this way, so it is folded into `taskWorktreePresent` and the re-adoption's `Gone` seam negates it.
  Batten now has one answer to whether a task worktree is there.
- **The promote refusal is judged from the board and the run's progress.**
  The start guard refused any run of a slug whose pair carries the beam mark unless the board held the slug as a project.
  A beam's Land prunes its done project entry, so the beam's own landing run read as a stopped promote at Worktree-Teardown.
  The guard's `Promoted` probe now answers true only while the board holds the slug as a task.
  The refusal is also skipped when the run's work reached its parent.
  A new `ReachedParent` seam decides that with the same probe `lym batten cancel` uses.
  That probe is now `hubChildReachedParent` in `internal/battencli/cancelcmd.go`, shared by both.
- **No refusal pair loops.**
  The promote refusal names `lym batten cancel <slug> --remove`, and the cancel refuses a run whose work reached its parent, naming `lym batten run`.
  The guard now names the cancel only where the cancel would go ahead, so the loop from issue #21 is gone.
- The `battencli` package doc states both rules.

## Design decisions

- A path that does not resolve as a worktree reads as absent whatever the resolver's cause, as the landed re-adoption already did.
  A real worktree whose anchor marker cannot be read therefore also reads as absent, and fabric's add then refuses the path by name.
- The promote check keys on the board holding the slug as a task, not on the run's recipe.
  An interrupted promote stops before its board write, so its slug is still a task; a landed beam's slug is gone from the board.

## To check by hand

- Issue #21 also reports `lym batten cleanup` failing with "not a git repository".
  Cleanup reads only the prime's records and scratch and never resolves a task worktree, so no change here reaches it.
  If it still fails over a stray directory, the error comes from elsewhere, possibly a command run from inside the stray directory.
- `taskWorktreePresent` now answers false for a stray path in every caller.
  These include the beam fork precondition, which then names `lym beam create` or `lym adopt` for the beam, and `ReleaseRunParentLock`, which then releases nothing.
  Check those wordings read right for a stray beam directory.
- The integration tests and an end-to-end landing of a real beam were not run by hand.
