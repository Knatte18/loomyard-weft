---
round: 4
cause: hard-cap
---
# Escalation — review segment `Discussion-Bouncer`, round 4

You are a one-shot fork of the parent session of a loom run.
The run's review segment `Discussion-Bouncer` for task `perch-budget-arbiter` has stopped at round 4 and is `awaiting` a decision.
The cause is `hard-cap`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYXHUB/_board/_lyx/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The round's review at `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter/_lyx/reviews/discussion/round-4-review.md`.
2. The round's ledger at `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter/_lyx/reviews/discussion/round-4-bouncer-ledger.md`.
3. The judge's verdict at `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter/_lyx/reviews/discussion/round-4-bouncer-verdict.md`.
4. The decision record at `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter/_lyx/discussion/decision-record.md`, when it exists.

The run's worktree is `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter`.

## What to do

Settle the question only when you can do it on the materials and the task's own record.
When you can:

1. Record each design call you make with `lyx loom decision add perch-budget-arbiter --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter`, when the decision record exists.
   Without one, name each design call in your report back to your parent session instead.
2. Record your decision on the segment with the one option this round has.
This round is at or past the perch's hard cap, so the segment can only be accepted:
   - `lyx loom circling accept perch-budget-arbiter` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
3. Resume the run with `lyx loom resume` in `/home/knatte/Code/loomyard-LYXHUB/perch-budget-arbiter`.

When you cannot settle the question, record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
