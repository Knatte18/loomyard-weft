# Loom tail: light fixers with test gates at Merge-In, Publish and PR-Align, no Merge-Review perch

`merge-in-light-fixer` · Note · enhancement, loom, burler, fabric

## Seen

After Webster-Review, loom's tail sends every failure back through heavy machinery built for other steps:

- Merge-In: `internal/mergeresolve` resolves textual conflicts; a verify failure after the merge goes back to Webster-Burler-Review.
- Merge-Review perch over how the task's code meets what the parent gained: ran in 73% of runs, about $0.96 per run; one of its fixes was a scope deviation PR-Align later rejected.
- Publish: a landing or publish gate failure also goes back to Webster-Burler-Review (about 38 verify failures at Merge-In and Publish in two weeks).
- PR-Align: a reject goes to PR-Rework, a new plan generation that walks Plan-Review, Webster, Webster-Review and the whole tail again; each rework plan was one fix card, about $4.80 and 23 min (audit `pr-rework-audit`).

## Idea (operator, 2026-10-22)

Webster-Review is Webster's; the tail gets its own light fixers instead.

- Drop the Merge-Review perch.
- Merge-In: its own light fixer, which resolves conflicts and fixes a failing verify, re-running the gated tests until they pass or its attempts run out.
- Publish: the same, its own light fixer behind the landing and publish gates, never a trip back to Webster-Burler-Review.
- PR-Align: a heavy rework with new plan and perches is overkill too (operator view, not a decision).
  Orch recommendation: a reject's findings go to the same kind of light fixer, then the gate, Publish and PR-Align again under a cap; PR-Rework with a new plan stays only for a reject that changes the goal.

## Open

- One shared fixer-with-gate row reused at all three sites, or one per site.
- Fixer model (Sonnet suits concrete failures and findings) and the attempt cap before the run escalates.
