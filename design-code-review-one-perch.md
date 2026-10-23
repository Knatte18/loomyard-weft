# One code review perch for darn and loom

`code-review-one-perch` · Task · enhancement, loom, burler

## Seen (orch, 2026-10-23)

Darn and loom each review the committed diff with their own perch: darn's `Darn-Review` (rows `Darn-Bouncer`, `Darn-Burler-Review`, `Darn-Burler-Fix`, keys `darn_*` in `darn.yaml`, rubric `loom-rubric-darn-review`) and loom's Webster-Review, which `loom-tail-review` renames `Code-Review` (keys `code_*` in `loom.yaml`, rubric `loom-rubric-code-review`).
Both run the same `BurlerReview` engine over the same subject; what differs is the answer key:

- darn: the task's board entry, amended by the record's `Added after Discussion` entries; the rest of the decision record is the writer's own claim, since the darn writer writes it unreviewed.
- loom: the decision record, approved by the Discussion perch, and the plan.

## Idea

One code review perch in both recipes, named `Code-Review` with one key set and one rubric, whose answer key the recipe sets: the board entry plus parent decisions in darn, the record and the plan in loom.

## Operator question (2026-10-23)

Why two code review perches, when both check that the code matches what was decided for the task?

## Operator proposal (2026-10-23)

The code perch checks the same thing in loom and darn: the code against a discussion record, and the code itself as code.
In loom it stays as Webster-Review checks today.
In darn the writer moves the task's description out of the board into a record file shaped like the one loom's Discussion writes, and adds its own decisions to it; the perch reads that file, not the board entry.

## Open

- The darn writer's own decisions are unreviewed, unlike loom's record, which the Discussion perch approves.
  The record must keep the board's copied scope apart from the writer's additions, so the perch checks each addition against the copied scope instead of taking it as the answer key; today's darn rubric does that by treating the record's other sections as the writer's claim.
