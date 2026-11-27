# batten cancel --remove of a beam clears the project's beam flag

`lym batten cancel <slug> --remove` on a beam's pawl run ended the run and removed the pair and its records, but the board project kept `"beam": true`.
The board refuses any payload that clears the flag, since a beam's life was meant to end only at `lym beam land`.
A beam cancelled before it landed was therefore stuck as a beam with no pair and no run to land.
The only way back to a plain project was to remove and re-create the entry, which changes its handle.

## What changes

- `boardengine.Board.EndBeam` is a new board write that clears a project's beam flag, together with the pawl recipe, owner machine and colour that only a beam carries.
  The project keeps its status, order and members, and the members' `project` field is untouched.
  It writes nothing for an absent entry or one that is no beam.
- `clearRunPhase` in `internal/battencli/cancel.go` calls the new `EndBeam` seam after it clears the claim, when `--remove` is set and the entry is a beam project.
  A failure there refuses with the cancel's re-run as the way forward.
  The report and the JSON envelope gain `beam_ended`, and the orch notice says the flag was cleared.
- With `--remove`, `refuseNothingLeft` counts a beam flag as something left to cancel.
  A beam already stuck this way, with nothing else left, is finished by running `lym batten cancel <slug> --remove` again, so an affected hub needs no hand edit.
- A cancel without `--remove` keeps the pair and the flag as before, since `lym beam create <slug>` restarts that beam.
- The board's refusal of `beam: false` now names the cancel with `--remove` as the second way a beam's life ends.

## The orch-stop warning

The cancel also warned `the beam's orch was not stopped: lym orch runs from the hub's prime worktree or a beam; "<slug>" is neither`.
The pawl run had ended before its Beam-Up step wrote the beam mark into the pair's origin record.
`orchcli.BeamDown` judges a worktree a beam by that mark, so it refused the pair.
No orch can run on such a pair, so the production `StopBeam` seam now reads the origin record and skips the stop when the mark is absent.

## Tests

- `TestEndBeam` (boardengine, tier 0) pins the cleared fields, the kept status and member, the no-op on a plain project, a task and an absent entry, and that `beam: false` upserts cleanly afterwards.
- `TestCancelRun` gains rows for a beam's `--remove` clearing the flag after the claim, a beam flag alone being left to clear, and a failed clear naming the re-run.
  The existing no-remove beam row now asserts the flag is kept.

The orch-stop skip sits in the production wiring in `cancelcmd.go`, which the fake-driven table does not reach, and no test pins it.

## Merge danger

**Door:** two-way
**Blast radius:** batten
