# Merge in from parent as a task of its own in a beam, in place of pawl's perch round

`parent-merge-task` · Note · enhancement, pawl, workspaces, drift, landing

## Seen (operator, 2026-10-31)

The pawl recipe, a beam's run, has a Code-Review perch of its own: when the beam lands, Parent-Align brings the parent in and the perch reviews the beam's whole diff (`contracts/recipes/pawl-recipe.yaml`).
The operator does not want pawl to have its own perch round: the beam's members are already reviewed by their own runs.

## Idea

Bringing the parent into a beam is a task of its own: a run with its own worktree inside the beam's project, "merge in from parent".
- It is a ready-made command the operator or the orch starts with one call, at any time, not only at landing.
- The run merges the parent into the beam, settles conflicts and gate failures with the merge fixer, and can run Drift.
- pawl then lands the beam without a perch round of its own.

## Open

- The command's name and where it lives (`lym beam ...`).
- Whether pawl starts such a run itself before it lands, or requires one to have landed since the parent last moved.
