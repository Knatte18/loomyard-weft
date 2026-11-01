# Merge in from parent as a task of its own in a beam, before pawl lands it

`parent-merge-task` · Note · enhancement, pawl, workspaces, drift, landing

## Seen (operator, 2026-10-31)

The pawl recipe, a beam's run, had a Code-Review perch of its own; the operator did not want it, since the beam's members are already reviewed by their own runs.
The perch is gone: pawl became a loom recipe with no review round (`pawl-loom-recipe`), and Parent-Align brings the parent in before Finalize and Beam-Down land the beam.
Nothing now checks that a beam has reached its project's goal before it lands: Beam-Await ends on a land request.

## Idea

Bringing the parent into a beam is a task of its own: a run with its own worktree inside the beam's project, "merge in from parent".
- It runs as darn with its perch on, not with review off (operator, 2026-11-01).
- A command on the beam, such as `lym beam merge-parent <beam>`, files such a task in the beam's project with one call, at any time, not only at landing; dobby starts it as any member (operator, 2026-11-01).
- The run merges the parent into the beam, settles conflicts and gate failures with the merge fixer, and can run Drift.
- pawl lands the beam without a perch round of its own (built by `pawl-loom-recipe`).

## Open

- The command's name.
- Whether its perch also reviews the beam against the project's decision record, so this is where the project's goal is checked before the beam lands.
- Whether pawl starts such a run itself before it lands, or requires one to have landed since the parent last moved.
