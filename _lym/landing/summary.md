# dobby: finish a paused run whose work already reached its parent

A batten run that stood `paused` after its work reached its parent branch was never finished.
Dobby gave a pass only to running runs, so Land and Worktree-Teardown never ran, and the task worktree and its terminal stayed up until an operator ran `lym batten run <slug>` by hand.
Seen on `cold-gate-fixer`: after `lym adopt root` took the hub over, the run stood paused at Run-Shed for hours with no notice to the orch.

Dobby now treats such a paused run as drivable.
It has nothing left to decide, so dobby gives it a full pass, which resumes it through Land and Worktree-Teardown.

## Changes

- `battencli.DobbyRuns` reads, for each paused run, whether its work already reached its parent, by the judgment `lym batten cancel` refuses on.
  The result is the new `RunView.ReachedParent`.
- A judgment that does not read leaves the run unreached, so it stays paused, and logs one Warn per failure episode instead of one line per tick.
- `dobbyengine` `planPass` accepts a run that is running, or paused with `ReachedParent` set.
  Such a paused run gets only full passes, never a continue pass.
- `lym batten pause` and `lym batten goto` help, the orch role contract and the `dobbyengine` and `beamcli` package docs name the exception.
  A pause or goto does not hold a run whose work reached its parent; `lym tender pause <slug>` does.

## Design decisions

- A paused run that a `lym tender pause` entry covers gets no pass and no notice while the entry stands.
  The full pass at the lift finishes it, as `planPass` already gave for covered runs.
  A pause entry is the operator's or hibernate's hold on all work, and Land pushes and tears down, so finishing under it would break the hold.
- `lym adopt` gains no step.
  After a takeover, the adopting machine's dobby finishes such a run through the same pass once no pause covers it, so a separate adopt step would duplicate it.
- No orch notice was added: finishing the run is the fix, and the run no longer stands waiting.

## To check by hand

- Not run against a real adopted hub: take over a hub holding a paused run whose work reached `main`, and confirm dobby finishes Land and Worktree-Teardown without a hand pass.
- A paused run covered by a pause entry gets no pass while it stands and a full pass at the lift.
  The change is meant to give this, but it is not pinned by a test (`covered-paused-run-no-pass-or-notice`).
- The new `ReachedParent` read in `battencli.DobbyRuns` has no test of its own; `dobbyengine` tests drive `planPass` with a preset `ReachedParent` (`dobbyruns-reachedparent-read-untested`).
