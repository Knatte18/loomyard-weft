# P5 Tapestry chains plan the next task while the one before still builds

Rendered 2026-11-28 12:45 UTC; every age below is as of then.
Under: [Board](README.md).

[`plan-ahead-chains`](design-plan-ahead-chains.md) · Age 51m · Owner `neve`

## Aim (operator, 2026-11-28)

A chain of tapestry tasks runs fully in series today, though the plan writer is the slowest row.
Starting the next task's planning before the one before has finished building gains more than parallel batches inside one plan,
which a card-graph study found buys about 1.6x at best (`docs/research/webster/parallel-batches-2026-10-17/report.md`).

## Parts

1. A recipe, `jacquard`, that runs the tapestry rows after Discussion and starts only from a stored decision record.
2. Plan-ahead on that recipe: a dependent task plans from its dependency's record and plan, parks before building, re-aligns after the dependency lands, then builds.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Jacquard, a recipe that runs the tapestry rows after Discussion and refuses an entry with no stored decision record** | [`recipe-build-from-sketch`](design-recipe-build-from-sketch.md) | motif | Discussion-Burler-Fix | neve | 1h<br>49m | shed, loom, batten, board |

## Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F2 |  | **A dependent task plans ahead from its dependency's record and plan, and waits only to build** | [`pipelined-chain-planning`](design-pipelined-chain-planning.md) | T6<br>F3 | 1h<br>2m | loom, batten, shed |
| F3 |  | **Released tasks start in a queue order the board shows and the operator can change** | [`release-order-queue`](design-release-order-queue.md) |  | 6m<br>2m | board, dobby |
