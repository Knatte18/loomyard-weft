# Tapestry chains plan the next task while the one before still builds

`plan-ahead-chains` · Project · feature, shed, loom, batten

## Aim (operator, 2026-11-28)

A chain of tapestry tasks runs fully in series today, though the plan writer is the slowest row.
Starting the next task's planning before the one before has finished building gains more than parallel batches inside one plan,
which a card-graph study found buys about 1.6x at best (`docs/research/webster/parallel-batches-2026-10-17/report.md`).

## Parts

1. A recipe, `jacquard`, that runs the tapestry rows after Discussion and starts only from a stored decision record.
2. Plan-ahead on that recipe: a dependent task plans from its dependency's record and plan, parks before building, re-aligns after the dependency lands, then builds.
