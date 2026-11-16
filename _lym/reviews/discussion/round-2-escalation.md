---
round: 2
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 2

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `weaver` has stopped at round 2 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F10` (MEDIUM): The board says the journal is pushed with the run so it outlives a move to another machine, while the implementer contract says the implementer never pushes and push-before-act-takeover owns pushes at shed transitions; only the parent can settle who pushes the journal mid-row Fix: Raise to the parent - should Weaver push after each batch as an exception to the implementer never pushing, should push-before-act-takeover push at points inside a producer, or is a push at the row's end enough? Record the answer as a Decision and drop the Open risks line and Q16.
- `F11` (LOW): The operator's 2026-11-15 decision says each batch is a producer, and the record reads it as one producer that runs every batch; only the parent can confirm that reading of the operator's words Fix: Raise to the parent - is one producer that runs every batch, with its stops going through shed's stop chain, an acceptable reading of the decision that each batch is a producer? Record the answer in D2 and drop the Open risks line and Q17.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D12 records a report found on disk before it attaches, so a strand still in its gate re-prompt loop is never attached, its failing gates are never seen, and the next batch spawns beside it Fix: In D12, replace the order "first records a finished report that was not yet recorded, then attaches for an attempt in flight" with - for every attempt state records in flight, Weaver first attaches through AttachGated with the attempt's report path and gates; the attached Wait settles a run whose report is already on disk under its gates; Weaver records the report only from the result that Wait returns, and records a report directly only when the attach answers not found. Update the D12 acceptance criterion to add - a crash while a strand is re-prompted on a failing gate attaches to that strand, and the batch is not recorded done until its gates pass.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add weaver`, then record the segment's outcome with exactly one of:

- `lym loom circling continue weaver` for a round over the amended record
- `lym loom circling accept weaver` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/weaver/_lym/reviews/discussion/round-2-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/weaver/_lym/reviews/discussion/round-2-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/weaver/_lym/reviews/discussion/round-2-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/weaver/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/weaver`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add weaver --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/weaver`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept weaver` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue weaver` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/weaver`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
