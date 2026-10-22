# A PR-Align reject goes to a light fixer row, not a new plan generation

`merge-in-light-fixer` · Note · enhancement, loom, burler

## Seen

A PR-Align reject goes to PR-Rework, which writes a new plan generation and walks Plan-Review, Webster, Webster-Review, Merge-In, Merge-Review, Publish and PR-Align again.
Audit `pr-rework-audit`: every rework plan was one fix card; a cycle cost about $4.80 and 23 min.

## Idea (operator, 2026-10-22)

- A reject from PR-Align goes to a new PR-Fix row: one fixer, as in a Burler fix round, fixing the code straight from the findings and the decision record.
  The findings are concrete, so a light model such as Sonnet suits it.
- Then verify, Publish and PR-Align again, under a bounce cap.
- PR-Rework with a new plan stays only for a reject that changes the goal: the operator's, or one the alignment agent marks as needing a new plan.
- Once PR-Fix exists, trial Merge-Review off (`merge_max_bounces: 0`): Merge-In's verify and PR-Align's check against landed parent commits cover it, and PR-Fix fixes what PR-Align finds.
