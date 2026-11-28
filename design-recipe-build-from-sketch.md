# A recipe that runs the tapestry rows after Discussion and refuses an entry with no stored decision record

`recipe-build-from-sketch` · Note · feature, shed, loom, batten, board

## Seen (operator, 2026-11-28)

The operator now seldom runs a full tapestry without a motif first.
A task raddle files from a Sketch already carries `discussed`, so its tapestry run skips Discussion;
which tapestry run does Discussion and which does not is read from a flag, not from the recipe named.

## Idea

A new recipe holds the tapestry rows after Discussion: Plan, Plan Perch, Webster, review and landing.
It runs only on an entry with a stored decision record, a task filed from a Sketch or a discussed entry, and refuses any other at start.
Tapestry then reads as motif plus this recipe, and the operator can say "run recipe X" with no doubt about whether Discussion runs.
Raddle names this recipe for the parts it files from a Sketch where it names tapestry today.

## Open

- The recipe's name.
- Whether plain tapestry stays as a recipe or becomes motif then this recipe.
