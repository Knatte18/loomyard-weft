# Plan and build become recipes of their own, so a planned task waits for its dependencies with no run held

`plan-and-build-recipes` · Note · feature, shed, loom, batten, board, dobby

## Seen (operator, 2026-11-28)

Motif already runs Discussion as a recipe of its own: it runs, stores its record on the entry and lands.
Plan and the implementer are rows inside tapestry, so a task planned ahead would have to park a run until its dependencies land.

## Idea

Split tapestry into three recipes, each run as its own cycle:

- X, motif: Discussion; stores the decision record in Bolt, since one record is often split into several tasks.
- Y, plan: Plan and Plan Perch for one task.
- Z, build: starts with a row that updates the plan for what landed since Y, then the implementer, review and landing.

A full run is X, Y, Z. One X often gives several Y, and each Y gives exactly one Z.

## Y and Z share one pair (operator, 2026-11-28)

- When Y ends, its task turns into a Z task with the same slug, so Z gets the same task branch and the same weft branch.
- The plan stays where it is written, in the pair's weft `_lym/plan`; it is never copied to Bolt, which would clutter it.
- Batten skips Worktree-Teardown when a Y run ends, so the local pair stays standing until Z starts in it.
- Nothing Z needs is trusted to origin: the pair is kept locally, since origin can be lost.
- A Y run ends with no run held: Z waits through the dobby's existing dependency wait, with no parked run and no wait row.

## Bounds

- One entry carries the task through X, Y and Z, its stage shown on the board, so the operator follows one entry and not three.
- Plan-ahead's bounds hold: Y of a dependent task runs only from an approved plan, and the choice to plan ahead is made up front.
- A standing pair with no run holds disk only, never a run place.
