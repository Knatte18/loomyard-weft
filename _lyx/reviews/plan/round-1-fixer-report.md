# Plan review round 1 — fixer report

## Changes

- **F1** (LOW, consistency), fixed as proposed.
  Card 3's `notOwnedRoute` way forward now opens with `fix the failure on the parent branch if it fails there too, then run "lyx loom goto --to Merge-In"`, followed by `or, if the task did cause it, run "lyx loom goto --to Darn-Write"`.
  Only the parent fix is conditional, which matches D2's order.
  The not-owned row of `TestDarnFailureRoute` now asserts that opening and that the Darn-Write goto comes after it.
- **F2** (LOW, scope), fixed as proposed.
  Card 9's Edit list gains `docs/overview.md`.
  An Intent sentence says the landing-tail paragraph keeps the Publish/PR-Gate/Finalize reroute line and adds Darn-Write's reroute to Merge-In.
  That reroute is taken when this run's failure record is present, the branch and worktree are unchanged since it, and the parent moved.
  The ImpactSummary now names the overview line.
- **F3** (LOW, design), fixed as proposed.
  Card 10 drops the "no parent tip" writer-runs row from `TestDarnWrite_Call`.
  The `rerouteReason` paragraph replaces "no tip counting as not moved" with a note: the seam always returns the local tip, so short of every tip being an ancestor, a failed tip or ancestor read is the only not-moved case.
  The self-check widened this note to include the failed ancestor read; see below.

## Self-check

- fix-claims: no findings.
- fix-affected-set: no findings, and every finding's set was checked.
- fix-proposal, LOW: card 10's new note named only a failed tip read as not moved, but a failed `IsAncestor` read also counts as no reroute, which card 10's own "a failed ancestor read" row pins. Fixed as proposed: "a failed tip or ancestor read".
- fix-consistency, LOW: `_lyx/plan/lens-review.md` still recorded a no-tip row for card 10. Fixed: that entry now notes that round 1's F3 later dropped the row as unreachable, and the rest of the record is kept as written.

## Disputed

None.

## Deferred

None.
