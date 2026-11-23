---
round: 1
cause: decision
---
# Escalation — review segment `Code-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Code-Bouncer` for task `agent-script-tmux-ate-memory` has stopped at round 1 and is `awaiting` a decision.
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

- `F4` (MEDIUM): Board item 1 asks for the refusal in every run agent's Bash, but the row is installed only when the spec is not interactive, so run sessions that can launch interactive (Discussion seat, PR session, drift judge, raddle, seat chair) keep raw tmux and script; the narrowing is the writer's own call, recorded nowhere Fix: Leave the code in place and defer this finding to the run's parent: recording or refusing the autonomous-only narrowing with lym loom decision add is the parent's call and lies outside the fixer's write surface.
- `F5` (LOW): Board item 2 leaves how narrow the tmux-tier opening is to a Discussion that never happened; the writer settled it in stencil prose alone, and nothing in lym gate test stops a run agent from running the whole tmux tier Fix: Leave the code in place and defer this finding to the run's parent, who records the prose-only narrowing with lym loom decision add or asks for a gate-side check.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): The tmux-script steer's way forward omits --tier, which board item 1 requires, so a call in its form runs tier0 and skips the tmux-tier test; it also points at a single-test form only the darn writer's instructions give Fix: Rewrite steerTmuxScriptDeny so its way forward is self-sufficient and names --tier, keeping it free of quotes and backslashes: after the outlives-every-memory-cap clause, say to write a tmux-tier test in the repository and run that one test through lym gate test with --tier naming the tier the hub runs tmux tests in, the test's package, and after -- a filter that selects it. Drop the clause in the form your instructions give for a single test. noticeTmuxScriptDeny follows from the steer. Regenerate internal/shuttleengine/claudeengine/testdata/settings-golden.json and rerun the engine tests that compare the steer.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, except that over a CONTINUE or CIRCLING verdict an open BLOCKING ledger entry of another class holds it: the round then runs another round, or inside the soft-cap window escalates again with cause budget, which only the parent accepts. An `accept` at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add agent-script-tmux-ate-memory`, then record the segment's outcome with exactly one of:

- `lym loom circling continue agent-script-tmux-ate-memory` for a round over the amended record
- `lym loom circling accept agent-script-tmux-ate-memory` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory/_lym/reviews/code/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory/_lym/reviews/code/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory/_lym/reviews/code/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add agent-script-tmux-ate-memory --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept agent-script-tmux-ate-memory` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue agent-script-tmux-ate-memory` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/agent-script-tmux-ate-memory`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
