---
round: 2
cause: decision
---
# Escalation — review segment `Code-Bouncer`, round 2

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Code-Bouncer` for task `autonomy-unattended-level` has stopped at round 2 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the budget is spent and the round cannot settle as an accept by itself:
  BLOCKING findings are still open, the round's live gate is red, or the soft-cap rule could not read its inputs.
  The detail below, when present, names which.
  A round whose review also holds `decision`-class findings reaches this cause when such BLOCKING findings are open,
  and its detail names those findings.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Settle each remaining finding yourself when it is a run-level call, and put it to the operator only when it is a design question, unless a recorded operator decision answers it.

- `F9` (LOW): The halt notices loom and batten post through NotifyParentHalt and NotifyRunFromHalt stamp through hubgeom.Autonomy and drop the refusal. A refused value reaches the orch only when the dobby builds one of its own stamped mails, so a run under beam.run_watch off whose halts come only from loom or batten is stamped false, and the orch never learns why. Fix: The parent chooses between two options. Accept the gap: D1 names the dobby as the poster, and the halt is still relayed under false. Then add one sentence to the runAutonomyStamps doc in internal/beamcli/dobbydeps.go saying that a refusal is told only once the dobby stamps one of its own mails about the run. Or build the missing part: have the dobby notice a refused value on its own tick for each listed run, whether or not it posts a mail, and post through the same runAutonomyStamps episode.

### BLOCKING findings of the round's review

- `F8` (BLOCKING): TestRunAutonomyStamps_PostsARefusalOncePerEpisode does not open with tiergate.Require naming its tier, as PATTERN-test-speed requires, and cmd/lym's TestTierTags_OneTierPerTestFile fails on it. Fix: In internal/beamcli/dobbyautonomystamps_test.go, make `tiergate.Require(t, 0)` the first statement of TestRunAutonomyStamps_PostsARefusalOncePerEpisode, before `t.Parallel()`, and add `"github.com/Knatte18/loomyard/internal/testkit/tiergate"` to the file's imports. Tier 0 fits: the test drives runAutonomyStamps in memory with no hub, git or tmux. Check with `lym gate test ./cmd/lym --tier 0 -- -run TestTierTags_OneTierPerTestFile`.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, except that over a CONTINUE or CIRCLING verdict an open BLOCKING ledger entry of another class holds it: the round then runs another round, or inside the soft-cap window escalates again with cause budget, which only the parent accepts. An `accept` at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

The task's scope closed before this segment, so no decision enters the record, and `lym loom decision add` refuses.
Rule each finding the decision record already asks for in scope by passing its id to `--in-scope` on the continue verb below; the next round's fixer then takes it.
File every other call as a board note for a new task with `lym board upsert` and kind note.
Then record the segment's outcome with exactly one of:

- `lym loom circling continue autonomy-unattended-level --in-scope <finding-id>`, the flag repeated per finding, for a round whose fixer takes the findings ruled in scope
- `lym loom circling accept autonomy-unattended-level` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level/_lym/reviews/code/round-2-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level/_lym/reviews/code/round-2-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level/_lym/reviews/code/round-2-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add autonomy-unattended-level --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
   When the verb refuses because the task's scope is closed, name the applied decision and its source in your report back instead,
   and handle each decision finding as the cause detail says for a closed scope.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept autonomy-unattended-level` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue autonomy-unattended-level` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/autonomy-unattended-level`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
