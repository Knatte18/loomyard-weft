# Raddle cuts darn families for size and concurrency

`raddle-darn-sizing` · Note · enhancement, shed, loom, darn

## Seen (darn-cost-measure, 2026-11-08)

`docs/research/loom/darn-cost-2026-11-08/report.md`:

- A darn costs about $4.48 fixed plus $1.43 per 1000 changed lines (r 0.31 over 279 darns).
- The 12 darns of the `fabric-test-speed` raddle split cost $14.16 per 1000 lines against $11.29 for the tapestry runs; 12 fixed costs are about $54 of the family's $58.53.
- The 12 darns ran 359 awake minutes in 163 calendar minutes, about 2.2 at a time, held back by a dependency chain.

## Decision (operator, 2026-11-11)

Smaller darns stay: each part gets its own run with its own review perches, which gives more control and more independent looks at the change.
A lower fixed cost per family is no reason to cut larger parts.
Raddle's split is not tuned for fewer, larger darns.

A large change runs as `arras`, then a project of darns (operator, 2026-11-08); the lean `satin` tapestry was weighed and dropped.

## Idea

Tune raddle's split for concurrency only: cut parts so the `depends_on` chain is short and parts with no dependency between them touch different packages, so more darns run at the same time.

Open: how raddle is told (its stencil, the split perch under `arras`, or both).
The recipe `arras` and its split perch (`Raddle-Bouncer`, `Raddle-Burler-Review`, `Raddle-Burler-Fix`, rubric `loom-rubric-raddle-review.md`) exist and already judge that recipe, profile and the task or family shape fit;
the rubric names no concurrency rule, which is what this note would add.
No full arras run has been driven end to end yet, so the first families give the figures.
