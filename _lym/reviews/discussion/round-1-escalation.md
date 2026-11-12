---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `testmap-checked-in` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F5` (MEDIUM): The board entry's Idea names an extra branch and junction in the weft repository and a raddle split into several tasks. D1 departs from both, and only the parent can confirm the departure. Fix: Raise to the run's parent both Open-risks lines. First, confirm D1's use of the existing weft branch instead of the board Idea's one extra branch and junction, or record the extra branch with lym loom decision add. Second, say whether raddle still splits the Sketch or the work stays one task. Once the parent answers, drop each answered line from Open risks.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D9 opens lym gate testmap to task worktrees but leaves a task's pass registered under the hub-wide runproc.HubOwner slot, so it blocks the prime dobby's pass and survives its pair's teardown, against PATTERN-child-processes Fix: In D9, decide and state how a task worktree's pass registers in internal/runproc. Register it under the task's slug, so pairteardown's runproc.End ends it before the worktree goes, per PATTERN-child-processes. Make the one-pass-at-a-time refusal and the dobby's TestMapRunning read count only passes of the same worktree, so a task's pass neither blocks nor is refused by the prime's. State this as D9's bound. Add an acceptance criterion that a pass in a task does not block the prime's dobby pass and is ended by its pair's teardown.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add testmap-checked-in`, then record the segment's outcome with exactly one of:

- `lym loom circling continue testmap-checked-in` for a round over the amended record
- `lym loom circling accept testmap-checked-in` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add testmap-checked-in --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept testmap-checked-in` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue testmap-checked-in` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/testmap-checked-in`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
