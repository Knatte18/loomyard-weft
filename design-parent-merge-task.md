# Merge in from parent as a task of its own in a beam, before pawl lands it

`parent-merge-task` · Note · enhancement, pawl, workspaces, reed, landing

## Seen (operator, 2026-10-31)

The pawl recipe, a beam's run, had a Code-Review perch of its own; the operator did not want it, since the beam's members are already reviewed by their own runs.
The perch is gone: pawl became a loom recipe with no review round (`pawl-loom-recipe`), and Parent-Align brings the parent in before Finalize and Beam-Down land the beam.
Nothing now checks that a beam has reached its project's goal before it lands: Beam-Await ends on a land request.
Bringing the parent into a beam is today done by writing a merge task by hand, as the `codex-engine` beam's two merge-main tasks were.

## Design (operator, 2026-11-01 and 2026-11-03)

- Bringing the parent into a beam is a task of its own: a darn run with its own worktree inside the beam's project, "merge in from parent", with its perch on.
- `lym beam merge-parent <beam>` files such a task in the beam's project with one call, at any time, not only at landing; dobby starts it as any member.
- The run merges the parent into the beam, settles conflicts and gate failures with the merge fixer, and can run Drift.
- Its perch reviews the merge alone; checking the project's goal before the beam lands is no part of it.
- pawl neither starts such a run nor requires one before it lands: pawl lands the beam without a perch round of its own (built by `pawl-loom-recipe`), and its Parent-Align brings the parent in as today.
- Two default buttons in reed's `beam` button group, each with a confirm prompt:
  - `merge-in` runs `lym beam merge-parent {beam}`.
  - `land` runs `lym beam land {beam}`.
- The prime has no parent, so the default `prime` group gets the beam group's other buttons without these two, instead of falling back to the beam group.

Related: note `protected-parent-landing`, landing on a parent the hub may not push to.
