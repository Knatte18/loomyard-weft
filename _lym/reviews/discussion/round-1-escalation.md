---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `testmap-upkeep-design` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F10` (MEDIUM): Board entry testmap-cross-package shows running at Darn-Write while this task's entry says the build tasks stay held until this record is approved; whether that run is held, and whether D13's changes reach its stored record before its writer writes code, is a parent question. Fix: Parent - confirm that the testmap-cross-package run is held at Darn-Write, or hold it, until D13's change for that entry is carried onto its stored record; then the writer drops the Open risks line and Q15 in the support log.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): The reconcile walk prunes retired keys from the hub base file before it applies reinterpretations, so D8's reinterpretation never sees test_map_every 0 and a hub that switched maps off gets upkeep switched on. Fix: In D8's last bullet, replace the configreg.Reinterpretation with the beam module's configreg Module.Migrate, which runs before the retired-key prune in configsync.ReconcileHubWideAt; state that Migrate rewrites test_map_every 0 to test_map_upkeep false and drops any other test_map_every value. Name the evidence in D8 - hubreconcile walk.go runs ReconcileHubWideAt (pruneSparse drops units a retired key covers) before applyReinterpretations. Change the Constraint's last clause to say test_map_every's 0 is carried by D8's migration, and change the D8 acceptance criterion's wording from reinterpretation to migration where it names the mechanism.
- `F2` (BLOCKING): Only an Affected verdict queues a declared test with no entry, so a gap left by an ended pass, a discarded round or a D4 outcome-entry drop is never queued again once the next plan reads Current, and the backlog does not drain on its own. Fix: In D1, state that under every verdict except Miss the plan queues each declared mapped test of the package that has no entry, spawn-marked tests excepted (D4); a Current verdict whose map has such a test re-stamps the map and queues those tests. In D4's outcome-entry bullet, state that dropping an outcome entry queues that test. Keep the D4 criterion that a yield-killed test is queued again, and add to it that the next pass after the one that ended still queues it.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add testmap-upkeep-design`, then record the segment's outcome with exactly one of:

- `lym loom circling continue testmap-upkeep-design` for a round over the amended record
- `lym loom circling accept testmap-upkeep-design` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add testmap-upkeep-design --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept testmap-upkeep-design` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue testmap-upkeep-design` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/testmap-upkeep-design`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
