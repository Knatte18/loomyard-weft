# Shed gains a row that repeats or unfolds into rows from a list, so a producer such as Weaver runs one step per batch

`shed-dynamic-rows` · Note · feature, shed, weaver

## Seen (operator, 2026-11-28)

Weaver runs its batches in its own Go loop inside tapestry's implementer row, because Shed could not step through them:
a recipe is a fixed file of rows and edges, and a producer's `Yield` ends the driver's loop (Weaver's decision record, Q1).
About 15–20% of Weaver's production code, its stop and re-entry glue, part of its resume and its run lock, therefore repeats in concept what Shed does.
Review rounds are a loop too, and the plan-ahead wait row of `pipelined-chain-planning` is a dynamic row.

## Idea

Let a recipe hold a row that repeats while its producer asks for another pass, or that unfolds into rows from a list its producer writes, such as a plan's batches.
Each pass or unfolded row then gets Shed's own stops, re-entry, resume, history and arbiter, and Weaver shrinks to a thin producer per batch.

## Bounds (operator, 2026-11-28)

- Shed is a vital part of lym, and changing its logic this deep can have fatal consequences.
- So the change is designed first, through motif, before anything is built,
  and it only adds: every existing recipe, row and edge runs as today, and a recipe without the new row kind takes none of the new code paths.
- Weaver stays as it is until the new row kind has proven itself; moving Weaver onto it is a task of its own.
