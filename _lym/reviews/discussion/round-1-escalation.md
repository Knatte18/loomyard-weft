---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `plan-split-producers` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding only the run's parent can settle.

### Decision findings

Each finding below is a question only the run's parent can settle.
Put each to the operator, unless a recorded operator decision answers it.

- `F3` (BLOCKING): The board requires Discussion to settle the content of the Parts-Write, part-writer and chair stencils with the operator before Plan; this autonomous run picked D10, D11 and D12 itself, and only the parent can say whether the operator approves them before Plan starts Fix: Route to the run's parent and ask whether the operator approves D10, D11 and D12 as written, or which changes they want, before Plan starts. Once the answer is recorded (lym loom decision add, or edits to D10 to D12), drop Open risks line 1.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): A failed writer does not halt the run with a Stuck today; the required seat's step end stops the chair, which mapOutcome turns into an ErrAgentDied error, so the run fails uncommitted with a reason that names neither the writer nor its findings Fix: In D5 or D6, add a decision that MultiLLMProducer maps a result whose step was ended by a seat (a SeatResult with EndedStep set) to shedengine.Stuck with a non-empty output pointer, so the commit decorator commits the part directories, and a Stuck reason naming the seat, its part and its gate findings path or Accept error. Cite internal/seatengine/engine.go join (the chair is stopped on ender.signal), internal/shedadapters/multillm.go mapOutcome (OutcomeDied becomes an ErrAgentDied error, and EndedStep is never read) and internal/loomshed/planwrite.go Call (commits only on a non-empty pointer) as the evidence. Rewrite D6's sentence to state that mapping, and add to the D5 or D6 acceptance bullet that a writer's exhausted gate or failed Accept leaves the run Stuck at Parts-Plan-Write with a clean tree and a reason naming the seat.
- `F2` (BLOCKING): lym loom join-parts is bounded only by the recipe's rows and the files existing, so any agent of the run at any later row (a Plan-Burler-Fix fixer, Webster) can run it and silently replace the reviewed plan's cards, dropping the chair's shared-spot cards and the review's fixes Fix: In D8, add a refusal that lym loom join-parts refuses unless the run's current row is the recipe's Parts-Plan-Write row (read from the run status's current producer and the recipe's rows, never a row name literal), with a way forward naming lym loom goto --to Parts-Plan-Write. Restate D8's Bound to say what the verb can now overwrite (only the plan directory, only while Parts-Plan-Write is the current row) and that a run past that row cannot reach it. Add to the D8 acceptance bullet that join-parts refuses while another row is current.
- `F3` (BLOCKING): The board requires Discussion to settle the content of the Parts-Write, part-writer and chair stencils with the operator before Plan; this autonomous run picked D10, D11 and D12 itself, and only the parent can say whether the operator approves them before Plan starts Fix: Route to the run's parent and ask whether the operator approves D10, D11 and D12 as written, or which changes they want, before Plan starts. Once the answer is recorded (lym loom decision add, or edits to D10 to D12), drop Open risks line 1.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add plan-split-producers`, then record the segment's outcome with exactly one of:

- `lym loom circling continue plan-split-producers` for a round over the amended record
- `lym loom circling accept plan-split-producers` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add plan-split-producers --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept plan-split-producers` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue plan-split-producers` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/plan-split-producers`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
