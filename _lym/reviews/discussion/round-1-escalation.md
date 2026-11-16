---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `adopt-build-resplit` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F2` (MEDIUM): D9 departs from the operator's stated basis for the stop, built on the run FSM's pause and cancel intents; only the run's parent can accept that departure Fix: Raise to the run's parent whether the operator accepts D9's hub stop without per-run pause or cancel intents, resting its pushed-state guarantee on D1's synchronous pushes? Record the answer as a decision for adopt-build-resplit, then drop or keep the Open risks line to match it.
- `F13` (LOW): The entry's prime-is-root-beam member says every reader of an empty parent_beam, naming the orch role, the dobby and the board verbs, moves to the named root; D8 keeps the dobby's and board verbs' reads on the empty slug, a departure from the operator's text raised to nobody Fix: Raise to the run's parent whether the operator accepts that only the printed parent_beam and its prose readers move to root, while the dobby's and board verbs' code reads keep the empty slug for the prime? Add the question to Open risks until the answer is recorded.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D1 retires only the teardown-lock respawn; the absorbing push lock's pending-flag respawn still pushes a flagged code side alone, so an old owner's code tip can land after a takeover and break the records-first order and D2's push-no-code-side guarantee Fix: In D1, add a bullet that the pending-flag respawn in internal/fabricengine/pendingflags.go (coalesceFlagged, respawnForeignFlags) no longer pushes a pair side alone. A flagged pair side is either left for the pair's next synchronous push or respawned through the records-first PushPairAnchored with D2's takeover read. Add an acceptance criterion that a code side flagged by a push whose lock wait gave up never reaches origin before its records side, and never after a takeover entry naming another machine.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add adopt-build-resplit`, then record the segment's outcome with exactly one of:

- `lym loom circling continue adopt-build-resplit` for a round over the amended record
- `lym loom circling accept adopt-build-resplit` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add adopt-build-resplit --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept adopt-build-resplit` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue adopt-build-resplit` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
