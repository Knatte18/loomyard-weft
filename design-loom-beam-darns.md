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
7. The rest is as today: Parent-Align, PR, landing, so `main` takes Task A whole.

## Why

`docs/research/token-audit-2026-10-22`: a loom run carries a fixed cost of about $23, mostly Plan and the review rounds, and the Plan writer reads about 250k of context before it writes (91% of its peak).
Darn's fixed cost is about $2.60.
Split this way, each darn reads only its piece once, plans and implements it, and is reviewed by its own perch; Discussion and the whole-task perch keep loom's scope and overall check.
It came out of discussing sub-planners for the Plan writer (note `plan-writer-context`): pieces small enough to plan and implement together are darn runs.

## Depends on

The beam setup: `orch-any-worktree` is landed (dd11ab09f: beams, pairs from a parent pair, `lyx beam create`, `land`; its `promote` was removed by `beam-start-and-ownership`, which also added `lyx beam start` and the `pawl` recipe), and `umbrella-daemon` is landed too (c2a03a885: dobby, a per-beam daemon that drives runs).
Earlier wording: `orch-any-worktree` (umbrella tasks spawning and landing from their own worktree) and `umbrella-daemon` (a daemon per beam that drives its runs).

## Open

- Contract precision: too loose and pieces collide, too tight and the pre-planner becomes today's Plan writer.
- A darn that finds its contract wrong stops and escalates to Task A instead of improvising; dependent pieces wait.
- What drives the darns inside the beam: dobby now starts and drives a beam's ready tasks (c2a03a885); open is how Task A's run waits for them.
- How large the whole-task perch's fixes may be before they mean a piece failed.
- Quality against full loom: each piece gets one perch instead of Plan and Webster review; measure follow-up fixes against loom runs.
- Cost: Discussion, pre-planner, N darns and the whole-task perch against a loom run of the same size.

## Review placement (operator, 2026-10-22)

- The pre-plan gets a perch, but the pre-plan is far smaller than today's plan format: pieces and their contracts, not cards.
- No perch per darn: a review per batch proved far too expensive in Millhouse, and a perch per piece is the same.
  The pieces run as darn with review off (`code_max_bounces: 0`, since `code-review-one-perch` made darn's perch the Code-Review perch), each still passing its own gate and tests; the whole-task perch after the last merge-back is the code review, as Webster's round over the whole diff is today.
  For scale, a darn perch costs about $1.30 per run in the audit window.
- The pieces run tests as they go, the way Webster's card gate runs after every card.

## Piece recipe `stitch` (operator, 2026-10-22)

The pieces run on their own recipe, `stitch`, derived from darn, not on a darn profile: rows go away, not only bounds.
Alignment per piece is pointless when the piece is a closed, synchronised share of an umbrella that is aligned whole.

- Kept: Preflight; the writer, on Sonnet, briefed by its pre-plan contract.
- Dropped: Code-Bouncer and the Code-Burler review/fix rows (review off), Describe and PR-Review.
  Since `loom-tail-review` (2026-10-24) darn has no Merge-In, Publish, PR-Align or PR-Gate rows; its Finalize merges the parent in under the lock.
- Finalize merges into the umbrella branch under the parent lock instead of landing on `main`.
- Friction-Reflect runs once per umbrella, not per piece.
- A `stitch` task runs only inside an umbrella; it cannot run alone.
- Estimate (orch, not measured): about $2-2.50 per piece against about $6 for a darn with review.

Operator view, not a decision (2026-10-24): once `stitch` exists, Codex could run the pre-planner, as a role apart from the Claude-written code (see note `codex-engine`).

## Operator view, not a decision (2026-10-23): darn itself as the piece implementer

Once `loom-tail-review` has landed, darn itself may serve as the pieces' implementer, since that task removes much of the run tail's ceremony, such as the pull request.
Whether `stitch` then stays a recipe of its own, or becomes darn with the tail rows a piece does not need turned off, is to be weighed against what `loom-tail-review` lands.

## Test depth of a piece (operator, 2026-10-23)

A piece run does not run the full suite a standalone darn runs before it lands.
The umbrella runs the full suite once, after every piece has merged back.
Which tiers a piece's own gates run is open; the gate key per recipe row is stated in `internal/gateslot`'s doc (`gate-runs-once`, landed): a piece's rows read `darn-write`, `code-burler-fix` and `finalize`.

## Pre-plan depth (orch, 2026-10-24, not decided)

- A coarse pre-plan leaves design to the pieces, so each piece needs an Opus writer.
- A detailed pre-plan carries the design, so Sonnet pieces may do, at the cost of a heavier pre-planner.
- Pieces cannot be counted on to run in parallel: `tools/plandag/plandag.sh` on `loom-tail-review`'s plan found 52 cards, a critical path of 20 and a mean parallelism of 2.6.
  A beam's gain is then mostly smaller review scopes, not wall time.
