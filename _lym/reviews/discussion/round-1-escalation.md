---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `wxb-plan-with` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F2` (LOW): Parent commit 7c23591, landed on main and on the parent beam branch after this worktree's base, already fixes this bug with a contradicting design; only the parent can say whether this bench arm redesigns from its base without reconciling Fix: Goes to the parent, not the fixer. The parent answers whether this arm, by design of the plan-writer-explorer-forks bench (frozen inputs from the reference run's base commit), redesigns the bug from base f98a1ba0 without reconciling with 7c23591; the answer is recorded with lym loom decision add, and the Open risks bullet at lines 93-95 is then dropped.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D2 claims the viewer ends on the beam overview window after the move, but no step of the mechanism selects it and reed's rules move the viewer elsewhere Fix: In D2, replace the sentence "The viewer ends on the beam overview window, as after today's replace." with a decided mechanism and its bound, for example that ensureRunsStrand, after a successful runs-strand add that followed a move which ended the viewed window, selects the runs strand's overview window through a reed op that selects a strand's window (name the op as new if reed has none), and that it selects nothing when the viewer was on another window; then reword the criterion at line 104 to that behaviour and say in Constraints which test tier pins it, since viewer selection is tmux-tier reed behaviour. If the writer drops the claim instead, delete the sentence and the criterion at line 104.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add wxb-plan-with`, then record the segment's outcome with exactly one of:

- `lym loom circling continue wxb-plan-with` for a round over the amended record
- `lym loom circling accept wxb-plan-with` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with`.

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
At cause `budget`, decide nothing: report back to your parent session which input could not be read.

When a recorded operator decision answers the question:

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add wxb-plan-with --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept wxb-plan-with` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue wxb-plan-with` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/wxb-plan-with`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
