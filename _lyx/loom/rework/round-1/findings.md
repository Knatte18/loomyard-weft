# PR-Gate review: discussion-multi-llm (PR #504)

## Finding 1 (MEDIUM): every advisor gets every question batch

`contracts/stencils/loom/loom-template-discussion-chair.md:14–15` tells the chair, with more than one advisor, to assign each advisor an area of the design to study, so each advisor sees only its own questions.
The operator's design (2026-10-10) is the opposite: the chair forms its question batches exactly as the solo writer does (each question with its alternatives), sends every batch to all advisors, and weighs their answers together, since each advisor may find a different weakness.

Fix:

- Replace the area assignment in the chair stencil: after Step 2 the chair sends each batch to every advisor, and settles each question after weighing all advisor answers (in autonomous mode; interactive mode stays as it is).
- Update the claims in `contracts/stencils/claims_test.go` that pin the old wording, if any, and the decision record's D5 and D6 sentences about areas are superseded by this review.

No other change is asked for; an operator-selectable area mode is a separate board note, not part of this rework.
