# Raddle cuts darn families for size and concurrency

`raddle-darn-sizing` · Note · enhancement, shed, loom, darn

## Seen (darn-cost-measure, 2026-11-08)

`docs/research/loom/darn-cost-2026-11-08/report.md`:

- A darn costs about $4.48 fixed plus $1.43 per 1000 changed lines (r 0.31 over 279 darns): the fixed part dominates, so fewer, larger darns cost less per line.
- The 12 darns of the `fabric-test-speed` raddle split cost $14.16 per 1000 lines against $11.29 for the tapestry runs; 12 fixed costs are about $54 of the family's $58.53.
- The 12 darns ran 359 awake minutes in 163 calendar minutes, about 2.2 at a time, held back by a dependency chain (`dotgit-worktree-reader` ahead of `fabric-test-folds` and others).

## Decision (operator, 2026-11-08)

A large change runs as `arras`, then a project of darns; a lean tapestry recipe with a solo Opus planner and Sonnet Webster (`satin`) was weighed and dropped, since its planner and Webster cost more than a darn of the same size.

## Idea (operator, 2026-11-08)

Tune raddle's split for darn families:

- Size: cut parts as large as one darn session handles well, instead of many small parts, so the family pays fewer fixed costs.
- Concurrency: cut parts so the `depends_on` chain is short and parts with no dependency between them touch different packages, so more darns run at the same time.

Open: what part size a darn handles well, and how raddle is told (its stencil, its perch on the split under `arras`, or both); the first `arras` families give the figures.

Built since (2026-11-09, `raddle-perch-mode`): raddle's perch mode exists.
A raddle run whose recipe rows hold a split perch (a Bouncer row over the raddle proposal) files in place of the operator's approval, and the perch accepts only one task or one project, never a beam or several parts as tasks.
The perch's review is therefore one place to tell raddle about size and concurrency, beside the stencil.

Built since (2026-11-09, `arras-recipe-perch`): the recipe `arras` and its split perch exist.
The perch is the rows `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`, reviewing the proposal against the decision record with the rubric stencil `loom-rubric-raddle-review.md`.
That rubric already judges that recipe and profile fit each part and that one task or a family fits the scope, but it names no part size or concurrency rule; those are what this note would add to the rubric or the raddle stencil.
The loom keys `split_*` (such as `split_max_bounces` and `split_reviewer_model`) tune the perch.
No full arras run has been driven end to end yet, so the first families still give the figures.
