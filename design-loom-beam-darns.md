# Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam

`loom-beam-darns` · Note · feature, loom, shed, batten, workspaces, burler

## Idea (operator, 2026-10-22)

A large task runs as a loom whose implementation is a set of darn runs inside its own beam:

1. Task A runs Discussion as today; Discussion stays the one agent that talks with the operator and decides scope.
2. A pre-planner breaks the decided scope into smaller pieces that together make up the whole scope.
   Each piece gets a contract: what it takes in, the behaviour it delivers, the names it exports to other pieces, and which pieces it depends on; not the inside, which the darn finds itself.
3. Each piece becomes a darn task with that contract and its part of the decision record and `relevant.md`, with `depends_on` between pieces.
4. The next producer turns Task A's worktree into a beam.
5. The darn tasks run from beam A as ordinary darn runs and merge back into Task A's branch, not `main`; pieces with no dependency between them run in parallel.
6. When every darn has merged back, a Task A-wide perch reviews the whole and makes small fixes.
7. The rest is as today: alignment, PR, landing, so `main` takes Task A whole.

## Why

`docs/research/token-audit-2026-10-22`: a loom run carries a fixed cost of about $23, mostly Plan and the review rounds, and the Plan writer reads about 250k of context before it writes (91% of its peak).
Darn's fixed cost is about $2.60.
Split this way, each darn reads only its piece once, plans and implements it, and is reviewed by its own perch; Discussion and the whole-task perch keep loom's scope and overall check.
It came out of discussing sub-planners for the Plan writer (note `plan-writer-context`): pieces small enough to plan and implement together are darn runs.

## Depends on

The beam setup on the board: `orch-any-worktree` (umbrella tasks spawning and landing from their own worktree) and `umbrella-daemon` (a daemon per beam that drives its runs).

## Open

- Contract precision: too loose and pieces collide, too tight and the pre-planner becomes today's Plan writer.
- A darn that finds its contract wrong stops and escalates to Task A instead of improvising; dependent pieces wait.
- What drives the darns inside the beam: the beam's daemon or orch thread, and how Task A's run waits for them.
- How large the whole-task perch's fixes may be before they mean a piece failed.
- Quality against full loom: each piece gets one perch instead of Plan and Webster review; measure follow-up fixes against loom runs.
- Cost: Discussion, pre-planner, N darns and the whole-task perch against a loom run of the same size.

## Review placement (operator, 2026-10-22)

- The pre-plan gets a perch, but the pre-plan is far smaller than today's plan format: pieces and their contracts, not cards.
- No perch per darn: a review per batch proved far too expensive in Millhouse, and a perch per piece is the same.
  The pieces run as darn with review off (`review_max_bounces: 0`), each still passing its own gate and tests; the whole-task perch after the last merge-back is the code review, as Webster's round over the whole diff is today.
  For scale, a darn perch costs about $1.30 per run in the audit window.
