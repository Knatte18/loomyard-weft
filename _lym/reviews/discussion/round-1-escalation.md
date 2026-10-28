---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a loom run.
The run's review segment `Discussion-Bouncer` for task `creel-hook-post` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding only the run's parent can settle.

### Decision findings

Each finding below is a question only the run's parent can settle.
Put each to the operator, unless a recorded operator decision answers it.

- `F12` (LOW): D6 leaves no channel for any creel post, program notices included, to reach a non-Claude session, which makes codex-engine's review-only first step waits on no creel work false and removes the typed cross-engine path creel-agents relies on; parent, accept this, or should those notes be updated when this lands? Fix: Raise under Open risks, for the parent: D6 removes every pane path for posts, so a Codex session, such as a Codex reviewer, gets no creel post, the LOOM decision notice included, until codex-engine adds hooks, and creel-agents' typed cross-engine message goes; accept, or have the operator update the codex-engine and creel-agents notes.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): The claim always hands out the first message whatever its size and a body may be 64 KiB, so a post over 10,000 characters reaches the session as a preview while creel journals it read; the Constraint that the 8 KiB budget keeps each field under the cap is false. Fix: In decision-record.md, add to D1 (or a new decision) how a single message whose line exceeds what fits in one hook field is delivered, with the evidence it rests on (internal/creelengine/verbs.go ClaimAtBoundary always hands out the first message; internal/creelengine/pool.go maxBodyBytes = 64 << 10). Pick one: lower maxBodyBytes to the 8 KiB boundary budget so post refuses a longer body with its existing write-it-to-a-file way forward, or have the claim deliver an oversized message as a fixed line naming its id, sender and size and the command that reads it in full, leaving it unread until that read. Rewrite the Constraints bullet on the 10,000-character cap so it states the bound the chosen rule gives, and add an acceptance criterion for one message larger than the budget.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add creel-hook-post`, then record the segment's outcome with exactly one of:

- `lym loom circling continue creel-hook-post` for a round over the amended record
- `lym loom circling accept creel-hook-post` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_board/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add creel-hook-post --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept creel-hook-post` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue creel-hook-post` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/creel-hook-post`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
