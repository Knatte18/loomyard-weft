# Jacquard, a recipe that runs the tapestry rows after Discussion and refuses an entry with no stored decision record

`recipe-build-from-sketch` · Note · feature, shed, loom, batten, board

## Seen (operator, 2026-11-28)

The operator now seldom runs a full tapestry without a motif first.
A task raddle files from a Sketch already carries `discussed`, so its tapestry run skips Discussion;
which tapestry run does Discussion and which does not is read from a flag, not from the recipe named.

## Idea

A new recipe, `jacquard`, holds the tapestry rows after Discussion: Plan, Plan Perch, Webster, review and landing.
It runs only on an entry with a stored decision record, a task filed from a Sketch or a discussed entry, and refuses any other at start.
Tapestry then reads as motif plus jacquard, and the operator can say "run jacquard" with no doubt about whether Discussion runs.
Raddle names jacquard for the parts it files from a Sketch where it names tapestry today.

## Name (operator, 2026-11-28)

The recipe is `jacquard` in code, config, verbs and records.
The operator calls it "jacq" in chat and text; the task adds `jacquard` to lym's glossary (`internal/heddleengine/glossary.md`)
and "jacq" to the nicknames in the repository's instructions, beside "Sketch" and "Merriam".

## Open

- Whether plain tapestry stays as a recipe or becomes motif then jacquard.
