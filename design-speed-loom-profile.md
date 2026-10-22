# Speed loom profile: Discussion, Plan and Webster, with review only of the end product

`speed-loom-profile` · Note · enhancement, loom, burler, config

## Idea (operator, 2026-10-22)

A loom profile that keeps Discussion, Plan and Webster, so the task is scoped, bounded and planned as in loom, but reviews only the end product: no Discussion, Plan or Webster review perch, only the review before landing.
More thorough and structured than darn, much cheaper than full loom.

## Evidence

`docs/research/token-audit-2026-10-22`: review loops are about a third of task spend, and Discussion and Plan review about $8 of a loom run's roughly $23 fixed cost.

## Open

- Whether profile settings alone can turn the perches off (bounce caps of 0), or whether that needs the row entry gate task `gate-runs-once` builds, instead of a recipe rewrite like darn's.
- Which review the end product gets: Merge review, or a Webster round over the whole diff.
