# Plan writer cost: a solo writer against twill-plan, and an excerpt cut

`plan-writer-context` · Note · research, loom, stencil

## Seen

The Plan writer is the costliest session type: about 18% of all spend in the token audit of 2026-10-22 (`docs/research/tokens/token-audit-2026-10-22`).
The 2026-11-08 re-measure (`docs/research/loom/writer-cost-remeasure-2026-11-08/report.md`, five tapestry runs) finds the Plan phase at $17.05 a run for the writer and its forks, 3.2 Plan rounds a run, and review and fix 51% of the phase, about $5.6 a round.
Exploration is 46% of the writers' cost, and the lookup subagents cost about what they keep out of the writers' context (`docs/research/turns/writer-lookups-measure-2026-11-04/report.md`).

## What stands

- Raddle splits a Sketch into parts before any plan, and the arras recipe is planned to run Motif and raddle in a row, so the Plan writer cost is avoided, not cut, for work that goes that way.
  Plan cost grows too slowly with size to need sub-planners per part; that idea is dropped.
- Discussion stays a separate step; folding it into the Plan writer would add turns to the costliest session (operator, 2026-10-25).
- A code digest instead of raw reads is rejected, and so are lens forks as fresh agents (that is the perch).
  The writer's read-back of the plan files after `fork0` is needed and stays.
- Split writing exists as the opt-in producers `Parts-Write` and `Parts-Plan-Write`, behind the tapestry flag `plan_split`, off by default; the benches (`docs/research/loom/plan-split/`) say keep it off.
- `relevant.md` works as a floor, not a boundary: 85% of the files it names are read, but only 28% of the files read are named (`docs/research/loom/plan-writer-audit-2026-10-24/`).

## Still open

1. A solo writer against twill-plan: compare twill, solo Opus high and solo Fable on Plan cost, Plan rounds and Plan wall time (`lym report timing`) per run.
   The operator sees strong signs that the twill writer makes a far better one-shot plan in shorter time, and time counts as well as tokens, so the bench may only confirm it.
   Too few runs have landed since the full twill fan to judge.
2. The excerpt cut: the first exploration fan answers with the lines needed, so the writer reads excerpts instead of whole files.
   The 2026-11-08 re-measure names it, with the review cost, as the larger pool; the most lookup context kept out of the writers could be worth is $5.95 of $46.94 over the 8 sessions measured.
