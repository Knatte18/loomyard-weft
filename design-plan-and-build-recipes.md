# Plan and build become recipes of their own, so a planned task waits for its dependencies with no run held

`plan-and-build-recipes` · Note · feature, shed, loom, batten, board, dobby

## Seen (operator, 2026-11-28)

Motif already runs Discussion as a recipe of its own: it runs, stores its record on the entry and lands.
Plan and the implementer are rows inside tapestry, so a task planned ahead would have to park a run until its dependencies land.

## Idea

Split tapestry into three recipes, each run as its own cycle of the same task:

- X, motif: Discussion; stores the decision record on the entry.
- Y, plan: Plan and Plan Perch; stores the approved plan on the entry the way motif stores the record, and lands.
- Z, build: starts with a row that updates the plan for what landed since Y, then the implementer, review and landing.

A full run is X, Y, Z. One X is often split into several tasks, so one X gives many Y, and each Y gives exactly one Z.
Y finishes and ends its run, so a planned task holds no run place while it waits;
Z starts through the dobby's existing dependency wait, so no parked run and no wait row are needed.

## Bounds

- The plan must outlive Y's pair: Y writes it to its weft branch, which the pair's teardown removes, so Y stores it on the board entry or in Bolt as motif does the record.
- One entry carries the task through X, Y and Z, its stage shown on the board, so the operator follows one entry and not three.
- Plan-ahead's bounds hold: Y of a dependent task runs only from an approved plan, and the choice to plan ahead is made up front.
