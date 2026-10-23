# One code review perch for darn and loom

`code-review-one-perch` · Note · enhancement, loom, burler

## Seen (orch, 2026-10-23)

Darn and loom each review the committed diff with their own perch: darn's `Darn-Review` (rows `Darn-Bouncer`, `Darn-Burler-Review`, `Darn-Burler-Fix`, keys `darn_*` in `darn.yaml`, rubric `loom-rubric-darn-review`) and loom's Webster-Review, which `loom-tail-review` renames `Code-Review` (keys `code_*` in `loom.yaml`, rubric `loom-rubric-code-review`).
Both run the same `BurlerReview` engine over the same subject; what differs is the answer key:

- darn: the task's board entry, amended by the record's `Added after Discussion` entries; the rest of the decision record is the writer's own claim, since the darn writer writes it unreviewed.
- loom: the decision record, approved by the Discussion perch, and the plan.

## Idea

One code review perch in both recipes, named `Code-Review` with one key set and one rubric, whose answer key the recipe sets: the board entry plus parent decisions in darn, the record and the plan in loom.

## Operator question (2026-10-23)

Why two code review perches, when both check that the code matches what was decided for the task?
