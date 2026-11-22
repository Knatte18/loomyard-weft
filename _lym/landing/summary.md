# battencli: commit the start rows without a push to origin

Since `71d6ee1a0`, every batten transition pushes its records to origin before the run acts.
The start rows from Seed-Child to Run-Shed each waited for that push, about 10 to 20 seconds apiece.
Measured from the `batten: <row> -> running` commits on the records branches, a run's start went from 1 to 13 seconds before 2026-11-18 08:23 to 40 to 70 seconds after.

## What changes

A batten run's uninterrupted start commits each row from Seed-Child to Run-Shed and pushes none of them.
The first later transition's push carries those commits.
Every later transition keeps its push.

A start row still pushes as any transition does in these cases:

- it resumes after a pause or halt, since another machine may take the run over there and its pushed state should be current;
- the status-commit marker is missing or unreadable;
- the previous push failed or was cut off.

A fresh run's seeding of the absent status now commits the Worktree-Create running row through the status-commit seam.
That commit is not a start-row skip, so it pushes, and it is the one push the start makes.
A failed commit there surfaces as a bootstrap failure; the seeded status is durable and the next run resumes it.

A machine that adopts the run before the first later push reads it at its last pushed row and runs the start rows again.
The starting machine's next push meets that takeover and hands the run over, as it does for any takeover.

## Design decisions

- `commitStatusMarker` gains `push_pending`, true from a commit until its push has landed or been left owed.
  A marker left pending, by a failed push or a crash between commit and push, makes the next start row push, so a skipped push is retried on resume.
  A marker written by an older binary has no such key and reads as not pending.
- The start core takes no network-gate admission, since it makes no network call.
- `statuscommit` itself is unchanged in behaviour: a caller that defers publication passes a `Push` that does nothing.
- `PATTERN-run-fsm`, `PATTERN-pair-remote` and `PATTERN.md` state the exception to "a transition commits and pushes before the run acts".

## To check by hand

- The start was not timed after the change; compare a fresh run's `batten: <row> -> running` commit times against the figures above.
- `fresh-run-seed-child-still-pushes`, in `internal/battencli/arm.go`: a fresh run still pushes once, at the seeded Worktree-Create commit, before Seed-Child.
  Check that this single push is the intended reading of "the start rows act without a records push".
- `failed-start-push-not-retried-on-resume`, in `internal/battencli/commitstatus.go`: check that a start row resumed after a failed or cut-off push pushes again, through `push_pending`.
- `start-branch-uncovered-behaviours`, in `internal/battencli/commitstatus_test.go` and `predrive_test.go`: check that the start branch's cases are all covered, including a missing marker and a marker from an older binary.

## Merge danger

**Door:** two-way
**Blast radius:** batten
