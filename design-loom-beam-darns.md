# stitch: a loom recipe that splits a large task into darn runs inside its own beam

`loom-beam-darns` · Note · feature, loom, shed, batten, workspaces, burler

## Idea

A large task runs on `stitch`, a stand-alone loom recipe that covers the whole course: it decides the scope, splits it into pieces, and has the pieces built as darn runs inside a beam that its own worktree becomes (operator, 2026-10-31).
A task gets `stitch` on the board before its run starts; switching a running tapestry to `stitch` after Discussion is a later idea, not this one.

1. Preflight, as in every loom recipe.
2. Discussion-Write, as tapestry's: the one agent that talks with the operator and decides scope.
3. Pre-Plan: an agent splits the decided scope into pieces that together make up the whole scope, and writes them to a file.
   Each piece gets a contract: what it takes in, the behaviour it delivers, the names it exports to other pieces, and which pieces it depends on; not the inside, which the darn finds itself.
   The pre-plan gets a perch, but it is far smaller than today's plan format: pieces and their contracts, not cards.
4. A Go row with no agent, like pawl's Record-Write, turns the task's board entry into a project and files each piece as a darn task in it, with its contract, its part of the decision record and `relevant.md`, and `depends_on` between pieces.
   No agent creates a task, so the rule that only the operator decides what becomes a task holds: the operator approved the split by releasing the `stitch` task, and the pieces come from a reviewed pre-plan.
   The row may wait for the operator's go-ahead on the pieces first.
5. A row runs pawl as an inner shed run in the same worktree, the way batten runs its recipe through `battenshed.NewInnerRun`.
   Pawl's Record-Write finds the record written and adds nothing, Beam-Up brings up the beam's dobby, which starts the darn tasks already filed, Beam-Await waits until every piece has landed in the beam, and Parent-Align, Finalize and Beam-Down land the beam in its parent.
6. `stitch` ends when its inner pawl run ends.

New pieces: the Pre-Plan producer, its format, validator and perch; the filing row; Beam-Await ending when every member has landed rather than on a land request; and the whole-task review below.

## Why

`docs/research/token-audit-2026-10-22`: a loom run carries a fixed cost of about $23, mostly Plan and the review rounds, and the Plan writer reads about 250k of context before it writes (91% of its peak).
Darn's fixed cost is about $2.60.
Split this way, each darn reads only its piece once, plans and implements it, and Discussion and the whole-task review keep loom's scope and overall check.
It came out of discussing sub-planners for the Plan writer (note `plan-writer-context`).
Whether it pays at all is open (operator, 2026-10-31), so the first step builds nothing: a test project done by hand with the projects and beams that exist.
A large task gets its Discussion, its pieces are filed by hand as darn tasks with review off in a beam project, and the whole course's tokens are measured against tapestry runs of the same size from the token audit.
The recipe is built only if the test shows tokens to save.

## Review and tests

- No perch per piece: a review per batch proved far too expensive in Millhouse, and a perch per piece is the same.
  The pieces run as darn with review off (`code_max_bounces: 0`), each passing its own gate and tests.
- One review over the whole task's diff after the last piece lands is the code review, as Webster's round over the whole diff is today; where it sits, in pawl before Parent-Align or as a row of its own, is open.
- A piece does not run the full suite before it lands; the beam runs it once, after every piece has landed.
  Which tiers a piece's own gates run is open; the gate key per recipe row is stated in `internal/gateslot`'s doc.

## Pre-plan depth (orch, 2026-10-24, not decided)

- A coarse pre-plan leaves design to the pieces, so each piece needs an Opus writer.
- A detailed pre-plan carries the design, so Sonnet pieces may do, at the cost of a heavier pre-planner.
- Pieces cannot be counted on to run in parallel: `tools/plandag/plandag.sh` on `loom-tail-review`'s plan found 52 cards, a critical path of 20 and a mean parallelism of 2.6.
  A beam's gain is then mostly smaller review scopes, not wall time.

Operator view, not a decision (2026-10-24): Codex could run the pre-planner, as a role apart from the Claude-written code (see note `codex-engine`).

## Open

- The choice between `stitch` and tapestry is made before the run starts, as darn against tapestry is today, while only Discussion reads the code that shows whether the task splits into pieces with clear seams (operator, 2026-10-31).
  `stitch` fits a large task over many packages that splits into parts with clear boundaries; tapestry fits a change woven tight, whose contracts would be as large as a full plan.
  Measure how often the choice is right; switching after Discussion is the fix if it is often wrong.
- Contract precision: too loose and pieces collide, too tight and the pre-planner becomes today's Plan writer.
- A darn that finds its contract wrong stops and escalates to the `stitch` run instead of improvising; dependent pieces wait.
- How large the whole-task review's fixes may be before they mean a piece failed.
- Quality against full loom: measure follow-up fixes against loom runs.
- Cost: Discussion, pre-planner, N darns and the whole-task review against a loom run of the same size.
