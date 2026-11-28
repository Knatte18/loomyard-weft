# P5 Tapestry chains plan the next task while the one before still builds

Rendered 2026-11-28 15:33 UTC; every age below is as of then.
Under: [Board](README.md).

[`plan-ahead-chains`](design-plan-ahead-chains.md) · Age 3h · Owner `neve`

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
| T9 | **Jacquard recipe holds tapestry's rows after Discussion, tapestry becomes the Motif and Jacquard blocks, and a Record-Seed row seeds the stored record** | [`jacquard-recipe`](design-jacquard-recipe.md)<br>[Sketch](decision-jacquard-recipe.md) | darn · solid | Describe | neve | 1h<br>1h | shed, loom, testkit |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Batten, loom verbs and the report admit jacquard where it walks tapestry's path, and cost cells count a discussed tapestry run as jacquard** | [`jacquard-recipe-lists`](design-jacquard-recipe-lists.md)<br>[Sketch](decision-jacquard-recipe-lists.md) | darn · light | T9 | 1h<br>1h | batten, loom, report, tools |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Jacquard refuses an entry with no stored record at batten's start and at a first lym loom start, and the decision verbs name the seed row** | [`jacquard-start-refusal`](design-jacquard-start-refusal.md)<br>[Sketch](decision-jacquard-start-refusal.md) | darn · light | T11 | 1h<br>1h | loom, batten |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Raddle names jacquard for the parts it files, and the glossary and docs name jacquard and tapestry's two blocks** | [`raddle-names-jacquard`](design-raddle-names-jacquard.md)<br>[Sketch](decision-raddle-names-jacquard.md) | darn · light | T13 | 1h<br>1h | parsers, loom, stencil, heddle, docs |

## Features (2)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| F2 |  | **A dependent task plans ahead from its dependency's record and plan, and waits only to build** | [`pipelined-chain-planning`](design-pipelined-chain-planning.md) | T19<br>F3 | 4h<br>1h | loom, batten, shed |
| F3 |  | **Released tasks start in a queue order the board shows and the operator can change** | [`release-order-queue`](design-release-order-queue.md) |  | 2h<br>2h | board, dobby |
