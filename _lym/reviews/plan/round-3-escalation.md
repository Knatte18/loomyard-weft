---
round: 3
cause: hard-cap
---
# Escalation — review segment `Plan-Bouncer`, round 3

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Plan-Bouncer` for task `fabric-core-push-origin` has stopped at round 3 and is `awaiting` a decision.
The cause is `hard-cap`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the budget is spent and the round cannot settle as an accept by itself:
  BLOCKING findings are still open, the round's live gate is red, or the soft-cap rule could not read its inputs.
  The detail below, when present, names which.
  A round whose review also holds `decision`-class findings reaches this cause when such BLOCKING findings are open,
  and its detail names those findings.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.



What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin/_lym/reviews/plan/round-3-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin/_lym/reviews/plan/round-3-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin/_lym/reviews/plan/round-3-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin`.

## What to do

Settle the question only when a recorded operator decision answers the envelope's question directly.
A recorded operator decision is one of:

- A passage of the task's board entry that the entry marks as an operator decision with a date, as `Operator decision (<date>)` does.
- A board note whose body states that it records an operator decision.
- A decision-record entry added with `--by operator` that is already in the committed decision record when the escalation arrives.
- A Discussion answer that the support log's `## Interview` shows the operator gave.

These never count: an `Idea` or agent-suggestion passage, an operator view not marked as a decision, a decision-record entry added with `--by arbiter` or `--by parent`, a writer's auto-pick, and an `## Auto-mode assumptions` entry.
`--by` is an unverified label, since only `--by arbiter` is restricted, so a `--by operator` entry counts only when it was in the committed decision record before the escalation.
Never settle a question whose answer widens the task's scope, and never accept over an open design-wrong finding unless a recorded operator decision covers that finding.
At cause `budget`, decide nothing: report back to your parent session what the cause names,
which the parent settles by accepting the round or ordering another one.

When a recorded operator decision answers the question:

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add fabric-core-push-origin --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
   When the verb refuses because the task's scope is closed, name the applied decision and its source in your report back instead,
   and handle each decision finding as the cause detail says for a closed scope.
2. On the ground of the operator decision you applied, record the segment's outcome with the one option this round has.
This round is at or past the perch's hard cap, so the segment can only be accepted:
   - `lym loom circling accept fabric-core-push-origin` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/fabric-core-push-origin`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
