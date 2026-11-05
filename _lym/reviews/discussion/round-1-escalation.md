---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `pawl-orch-landing` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding only the run's parent can settle.

### Decision findings

Each finding below is a question only the run's parent can settle.
Put each to the operator, unless a recorded operator decision answers it.

- `F7` (MEDIUM): The board entry asks that a push lock whose holder died is taken as stale, but such a flock is already free; whether lym should stop a live push left from before a restart, or the point drops, is the parent's call Fix: Parent decides whether issue

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D11's crash recovery and D10's goto bound rest on the dobby rewriting the land request, but the dobby's persisted land episode stops it, and the dobby-stop marker D11 keys on is already removed by the accept Fix: In D11, replace the sentence that starts the dobby when the dobby-stop marker stands and has the dobby write the land request again. State instead that Beam-Await's entry, on finding the hold named Finalize or a retired row with no land request, reads the tracked ready record for this slug. If the record stands, the entry runs the accept's re-check itself, with no land request, and then is Done or refuses as the accept does. If it does not stand, the entry lifts the hold and starts the dobby whether or not the marker stands. Give the same entry rule in D10's Bound for a goto back to Beam-Await while the ready record stands. Drop the claim that the dobby lands the beam again. Update D11's and D10's acceptance criteria to match. Name the dobby's persisted land episode (dobbyengine State.Land, cleared only on a non-passing reading) in Constraints as the fact this rests on.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add pawl-orch-landing`, then record the segment's outcome with exactly one of:

- `lym loom circling continue pawl-orch-landing` for a round over the amended record
- `lym loom circling accept pawl-orch-landing` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add pawl-orch-landing --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept pawl-orch-landing` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue pawl-orch-landing` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/pawl-orch-landing`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
