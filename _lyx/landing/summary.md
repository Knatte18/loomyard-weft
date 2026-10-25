# An adopt runs the finalize gate on the adopting machine

`lyx adopt` now ends by starting the hub's `finalize` gate (`lyx gate test --gate finalize --wait`) in the prime worktree, so a break a machine change brings to the landed code (Ubuntu 26.04, Ubuntu 24.04, Windows 11) shows at adopt time instead of inside the next run's gate.
This replaces the `ubuntu-2404-floor` note: there is no PATTERN entry for an Ubuntu floor and no CI job.
What fails is fixed on the machine that shows the break.

## What changes

- `internal/adoptcli`: a new last step, after the dobbies, behind a new `adoptDeps.StartSuite` seam.
  It runs only when the call recorded this machine as owner of at least one entry or of the prime; a refused call or one that took nothing over starts nothing.
- The envelope gains `suite` (`pid` and `log`) when the suite started, or `suite_warning` when the spawn failed.
  The warning names `lyx gate test --gate finalize` in the prime as the way forward and leaves every slug's outcome as it was.
- The suite's output goes to `<hub>/_board/.lyx/adopt/suite.log`, truncated at each start.
  The directory name is the new `lyxdirs.AdoptDirName` token, reached through `fabricengine.AdoptScratchDir`.
- The `lyx adopt` long help, the `internal/adoptcli` package doc and the adopt entry in `docs/overview.md` describe the step.

## Design decisions

- **The full suite is the `finalize` gate key.**
  It is the hub's own statement of the whole-tree check that guards a landing, and a hub that retunes `gates.finalize` retunes the adopt suite with it.
  A tier already green for the clean tree is carried and runs nothing, as for any gate call.
- **One suite per `lyx adopt` call, in the prime.**
  Adopting the prime and two beams in one call starts one suite.
  Two separate calls each start one; the gate slots serialize them, and the second carries a tier the first left green only if it starts after the first finished.
- **A red suite only reports.**
  The suite runs after the owner write and the resumes, which have already taken effect, so it never refuses or undoes anything.
- **The suite runs detached and the adopt returns at once**, through `proc.Detach`, because the suite runs for many minutes.
  Nothing reads the log on the operator's behalf; the gate's own run records hold each tier's result.
- The spawn is logged through `internal/logger`, and the suite holds the gate slots through `lyx gate test`.
  `startSuite` starts nothing under `go test`, as the other detached spawns do.

## Not changed

No change to `lyx gate test` itself, and no creel notice for a red suite.

## To check by hand

The detached spawn is not exercised by the tests, which fake the seam.
On a real hub, run `lyx adopt` for an entry owned by another machine and check that the envelope carries `suite` and that `suite.log` fills.
A suite red from a flaky test reports a break the machine change did not cause; rerun the gate.
