# Dobby finishes a paused run whose work already reached its parent

A batten run whose child already merged into the parent branch has only its Land and Worktree-Teardown rows left.
Dobby gave a paused batten run no pass, so such a run could stand paused for hours with its task worktree and terminal still up, until someone ran `lym batten run <slug>` by hand.
That is what happened to `cold-gate-fixer` after `lym adopt root`: its work was on `main`, the run stood paused at Run-Shed with "pause requested", and nothing finished it.

## What changes

- `dobbyengine.RunView` gains `ReachedParent`, read for a paused run only.
  `planPass` treats a run as drivable when batten's state is running, or paused with `ReachedParent` set.
  Such a paused run gets only full passes, the same `lym batten run <slug>` a hand resume runs, and gets none while a pause entry covers it, bar the full pass at the lift.
  A held (adopt hold) run, a cancel-marked run, a pass in flight and a held driver lock still give no pass, as for a running run.
  A halted run (blocked, failed, awaiting) gets no pass, whatever its work's state.
- `battencli.DobbyRuns` fills `ReachedParent` for each paused run through `hubChildReachedParent`, the same judgment `lym batten cancel` refuses on: the child past Finalize's parent-side merge, a Finalize done in the child's history, or batten past Run-Shed with the child done.
  A judgment that does not read logs one Warn and leaves the run unreached, so the run stays paused and the tick goes on.
- The dobbyengine and beamcli package docs and the orch role contract say so.
  The orch role now names `lym tender pause <slug>` as the way to hold such a run, since `lym batten pause` no longer holds it.

## Design decisions

- The fix sits in dobby only.
  The board entry's second idea, that `lym adopt` finish landed runs itself, is left out: a run adopt holds is resumed by its subject's green adopt check, which starts a full pass of each lifted run, and a run left paused outside the hold, whether before or after the adopt, is now finished by its parent's dobby.
- A paused run covered by a pause entry is left alone and gets no notice.
  A pause entry is an explicit hold, and its lift gives the run the pass.
- Every batten paused state counts, a fired `pause --before`/`--after` condition included, since the entry states that a landed run has nothing left to decide.

## Risks

- A pass that exits 0 but leaves the run paused again would get another full pass on each probe tick, bounded by the probe interval; a pass that keeps failing raises dobby's existing failing-pass notice after three failures with one text.
- The reached-parent read runs on every dobby tick for each paused run.
  It reads the child's status file, and touches the child's settle records and git only when the child stands at Finalize.

## Tests

`TestPlanPass` gains rows for a paused run with and without `ReachedParent`: uncovered, covered, at a lift, with origin owed, cancel-marked, and a halted run with `ReachedParent` set.
The reached-parent judgment itself is pinned by the existing `TestChildReachedParent`.
