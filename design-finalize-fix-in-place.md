# Finalize settles a rejected fixer change in place and never sends the run back

`finalize-fix-in-place` · Note · enhancement, landing, shed, loom, darn

## Seen (operator, 2026-10-31)

Finalize still sends a run back in one case: when its merge fixer spent its cap, or made an over-cap change, and Finalize's own solo fix review does not approve that change.
tapestry's Finalize then reroutes to Parent-Align and darn's to Code-Bouncer (`on_reroute` in `contracts/recipes/tapestry-recipe.yaml` and `darn-recipe.yaml`), bounded by Shed's reroute cap.
The operator does not want a run bounced back to earlier rows, and a halt handed to the orch helps no one either, since the orch can do little with it: the run must settle it itself.

## Idea

Finalize settles a fixer change in place, under the parent lock, and never reroutes.
- A fix review that does not approve hands its findings to the fixer, which fixes them in place, and the review checks again: a small perch inside Finalize.
- The loop has a cap; at the cap Finalize escalates with the unresolved findings, as a review loop that ran out of rounds does, and the operator decides.
  The operator can accept the change as it stands, and the run then lands with the unresolved findings written into the change description, as `lym loom circling accept` does for a review loop.
- Parent-Align and Code-Review are not run again, and `on_reroute` goes from Finalize in every recipe.
