# Arbiter becomes a one-shot fixer that may edit and commit

The arbiter (`internal/shedarbiter`) used to give a verdict only: resume with a goto held to the halted row, or hand on to the run's parent.
A stop that needed a small edit therefore went to the orch or the operator, though the arbiter had read enough to make it.
Now the arbiter may settle the stop with one fix of its own, committed on the task branch, then resume the run.
The change is the arbiter's, for every recipe and for both loom's and batten's driver.

## What changes

- The arbiter stencil gets an `edit_scope` marker, filled with one of two new fragments.
  `loom-fragment-arbiter-fix` lets the arbiter fix the task's code, tests and docs in the task worktree and commit on the branch it found.
  Every commit ends with the trailer `Arbiter-Attempt: <run id>`.
  `loom-fragment-arbiter-no-fix` keeps the arbiter to a verdict, as before.
- The plan, the decision record and PATTERN entries still change only through the recipe's repair verbs.
  The arbiter never edits under `_lym/` or `.lym/`.
- `shedarbiter.Deps.Worktree` is a new seam; nil keeps the arbiter to a verdict, and its prompt says so.
  `internal/shedworktree` supplies the seam over fabric's read-only reads.
- The call records the worktree snapshot and the uncommitted paths in `attempt.json` at spawn.
  After the arbiter ends, it reads the commits made since.
  A resume is handed on with cause `precondition` when the arbiter left an uncommitted change (a new file included), switched branch or rewrote history.
- A resume after the arbiter's commits needs a goto target, so the run's own rows and gates check the fix before the run goes on.
  A resume with commits and no goto is handed on.
- The commits show in `envelope.json`, the `lym loom arbiter` output (`commits`), the stop report's attempt line and the settle records (`SettleRecord.Commits`).
  A hand-on's question names them, so the run's parent keeps or reverts them.
- `loomcli` wires the seam over the run's task worktree.
  `battencli` does the same, per call, through `withTaskWorktree`.
- `docs/overview.md`, the `loomcli` and `shedarbiter` package docs, and batten's recipe description are updated.

## Design decisions

- One arbiter behaviour everywhere: batten's arbiter gets the same seam, edit rights and instructions as loom's, with no batten-specific instructions.
- Batten's arbiter edits in the task worktree, never the hub's prime, whose HEAD and tree move under other writers.
  Where the task worktree is absent (before Worktree-Create, after Worktree-Teardown), that call gets no seam and gives a verdict only, and its prompt says so.
  Presence is read on each call.
- The check is the run's own gates, not a gate of the arbiter's: it must name a goto at or before the first row that checks the change.
- The arbiter may not push, amend, rebase, reset or switch branches; the driver's comparison with the spawn snapshot enforces this after the fact, not before.

## To check by hand

No regression check ran, and the end-to-end path was not run against a live run.

- Run a loom or batten run to a stop an arbiter can fix.
  Confirm the commit carries the `Arbiter-Attempt` trailer, the settle record lists it and the goto re-runs the checking rows.
- Batten stencil wording: the arbiter's recipe description and `docs/overview.md` now say the arbiter may fix while the task worktree exists.
  Check that they read consistently with the new fragments, since reviews flagged this inconsistency.
- Batten seam: check that `withTaskWorktree` names the task worktree and never the prime.
  Also check that the no-fix prompt is used while the worktree is absent.
- The arbiter stencil's "What you may change" section: check that it no longer contradicts the fix fragment, which a review flagged.
