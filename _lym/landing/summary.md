# batten cancel --remove of a beam clears its beam flag

`lym batten cancel <slug> --remove` on a beam now ends the beam's life.
It clears the project's beam flag and leaves a plain project, with its handle and its members unchanged.

Before this change the cancel removed the pair and records but left `"beam": true` on the board.
No write could clear the flag, and nothing was left to land.
The only way out was to remove and recreate the project.

Closes #259

## Behaviour

- With `--remove`, a task member with neither a pair on this machine nor a run status no longer refuses the cancel.
  It keeps its `project` field.
  These still refuse: a member with a pair or a run status, a member that is itself a beam,
  and a pair forked from the beam.
- Without `--remove`, the beam keeps its flag and every open member still refuses.
- A beam flag that is all that is left of a slug now counts as something to cancel under `--remove`.
  A beam already stuck this way is cleared by running `lym batten cancel <slug> --remove`.
- A pair with no beam mark in its origin record no longer gets an orch stop,
  which removes the "orch was not stopped" warning.
- The JSON envelope has a new `beam_ended` key, and the orch notice names the cleared flag.
- The upsert refusal for `beam: false` now also names `batten cancel --remove`.

## Design decisions

- The new `Board.EndBeam` clears `beam`, `recipe`, `owner_machine` and `beam_color`.
  It keeps the project's status, order and members.
- `EndBeam` clears an owner through the ordinary board write and detached sync, not through `SetOwners`.
  The `boardengine` package doc and `PATTERN-owner-machine` now name it as the one other write of an owner.
  The reason given is that the cancel has already refused a beam another machine owns.
- A sub-beam member stays a blocker even when it looks idle, because it can be open on another machine
  with no local pair and no run status.
- The flag is cleared after the records push, so a failed clear does not skip the push.
  The error names the same `--remove` command to run again.

## To check by hand

- `EndBeam` is a second writer of `owner_machine` (`internal/boardengine/beamend.go`).
  It does not pull first or push in its own commit as `SetOwners` does.
  Check that the ordinary write and detached sync are enough.
- The merge danger of this change was understated as two-way with a radius that left out `boardengine`.
  Check the call below against the board fields `EndBeam` erases.
- The skip of the orch stop on an unmarked pair (`internal/battencli/cancelcmd.go`, the `StopBeam` seam)
  was found untested.
  `TestCancelIntegration_RemoveOfABeamLeavesAPlainProject` now asserts that no warning appears.
  Check that this test fails when the guard is removed.

## Merge danger

**Door:** one-way
**Blast radius:** board

A revert does not restore the beam fields that `EndBeam` has already erased and the board sync has pushed.
