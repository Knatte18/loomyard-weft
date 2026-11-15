---
round: 2
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 2

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `adopt-redesign` has stopped at round 2 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F12` (MEDIUM): D10 pushes the orch's whole conversation to the weft repository, which is public, and only the operator can allow that Fix: Parent asks the operator whether orch transcripts may go to the public weft repository. On a no, D10 names another place, such as a private repository, or drops the mirror; on a yes, the Open risks line goes.
- `F13` (MEDIUM): lym adopt --release lets the hub go on with failing tests, against the operator's decision that nothing happens until every failing test adopt found is fixed Fix: Parent asks the operator whether `lym adopt --release` may exist. On a no, D8 drops it and states how a hold whose adopter stopped is cleared; on a yes, the Open risks line goes.
- `F14` (MEDIUM): D8 keeps the orch up and lets it cancel runs during a hold, against the operator's decision of no orch action until adopt is done Fix: Parent asks the operator whether the orch may be up during a hold. On a no, D8 starts the orch only once the hold clears and names who reads the fix task's halts meanwhile; on a yes, the Open risks line goes.
- `F15` (LOW): D2 stops every run while origin is unreachable, and the operator has not said whether offline progress is wanted Fix: Parent confirms with the operator that no run may act offline; on a yes, the Open risks line goes.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): After a takeover, D4 kills the old owner's landing past its parent-side merge while D1 lets the adopter drive the same Finalize, so the carve-out holds on neither side Fix: Make one machine finish a landing past its parent-side merge after a takeover, and state it in D1, D4, D6 step 4, D8 step 3 and D9 alike. Recommended: the adopter finishes it, since D2 puts every transition on origin and a Finalize past its merge resumes from there. Then drop "and when another machine owns it" from D1's landing `act` line, drop the landing carve-outs of D6 step 4 and D9 in favour of D4's hand-over, and reword the `PATTERN-owner-machine` constraint and its declared change to say the owner finishes it. If the old owner finishes it instead, D8 step 3 writes no takeover entry for such a run, and D1 answers `wait` on the new owner until the batten record reads `landed`. Add an acceptance criterion: after a takeover, a landing past its parent-side merge is driven on exactly one machine.
- `F2` (BLOCKING): A pair push sends the code side before the records side, and the takeover entry is on records branches only, so the old owner's post-takeover code commits land on origin's code branch Fix: In D2, order a task pair's transition push records side first, and push the code side only once the records side has landed. In D4, a records push rejected over a takeover entry pushes no code side; its code commits go to the archive tag with the rest. State in D4 how the adopter's code push reads a code branch that still moved, and add an acceptance criterion: after a takeover, the old owner's next transition leaves origin's code branch unchanged.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add adopt-redesign`, then record the segment's outcome with exactly one of:

- `lym loom circling continue adopt-redesign` for a round over the amended record
- `lym loom circling accept adopt-redesign` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign/_lym/reviews/discussion/round-2-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign/_lym/reviews/discussion/round-2-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign/_lym/reviews/discussion/round-2-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add adopt-redesign --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept adopt-redesign` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue adopt-redesign` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/adopt-redesign`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
