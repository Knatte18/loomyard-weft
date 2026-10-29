---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a loom run.
The run's review segment `Discussion-Bouncer` for task `pr-review-row` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding only the run's parent can settle.

### Decision findings

Each finding below is a question only the run's parent can settle.
Put each to the operator, unless a recorded operator decision answers it.

- `F4` (BLOCKING): With pr_account empty or the operator's own account, GitHub refuses the operator's approval, so the board entry's first ending, the operator approves on GitHub, cannot happen; whether lym must require another pr_account, accept a merge on GitHub as the operator's ending, or offer a lym-side approval is a question only the parent can settle Fix: Raise to the run's parent the question the Open risks bullet asks, how the operator approves a pull request opened under their own account, with the three options it names. Record the answer through lym loom decision add, then drop the Open risks bullet and the open Q13 in the support log.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D10 keeps a wait marker of a kind the quiet check already reads, but none of the four kinds readAgentActivity counts fits PR-Review, so the record defers which marker exists to Plan and the quiet notice the operator ruled out goes off Fix: In D10, name the mechanism and its lifetime, settled from the code. The four live-wait kinds in `internal/battencli/wire.go` readAgentActivity are a non-paused verify marker (verifytree, live only during a gate), a parent-lock wait record (contradicts D9's no lock), a child status running at pawl's Beam-Await row (matched by row name in childAtBeamAwait) and a non-held shuttle wait marker (written only inside a shuttle Run's Wait, which D7's un-awaited session has not). State the chosen one, for example that readAgentActivity also reads a child status running at the PR-Review row, beside childAtBeamAwait, live for as long as the status reads running at PR-Review, and drop the words of a kind battenshed's quiet check already reads. Update the support log's Turn 4 and Q5 to match, and the D10 acceptance criterion to name the mechanism.
- `F2` (BLOCKING): D8's bound says D9's Done reads only the record's gated head, but D9's merged ending compares the merged head with the task head, so an ungated agent commit carried by the detached records push and merged on GitHub ends Done and lands through Finalize's already-landed path ungated Fix: Settle the merged-on-GitHub case and make D8 and D9 agree. Either make D9's merged Done require the merged pull request's head to equal the record's gated head and add a merged-with-an-ungated-head halt naming the head and the way forward, or rewrite D8's bound to say truthfully that a merge on GitHub lands whatever head the pull request holds, gated or not, and what bounds it. Add a D8 or D9 acceptance criterion for an ungated head merged on GitHub.
- `F3` (BLOCKING): The session's limit never pushes, never approves, merges or closes the pull request and never calls GitHub is only a stencil instruction; the session runs in the task worktree with git push and gh available, and the Decision states no enforced bound or the residual it lets through Fix: In D7, state what bounds the session. Either deny the escape on the launch spec, for example settings denying `git push` and `gh` through the shell-guard table, and say so as the bound, or state the residual plainly as a bound, that the session can push or merge on GitHub and what then catches it (F2's gated-head check). Add a D7 acceptance criterion for the chosen bound.
- `F4` (BLOCKING): With pr_account empty or the operator's own account, GitHub refuses the operator's approval, so the board entry's first ending, the operator approves on GitHub, cannot happen; whether lym must require another pr_account, accept a merge on GitHub as the operator's ending, or offer a lym-side approval is a question only the parent can settle Fix: Raise to the run's parent the question the Open risks bullet asks, how the operator approves a pull request opened under their own account, with the three options it names. Record the answer through lym loom decision add, then drop the Open risks bullet and the open Q13 in the support log.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add pr-review-row`, then record the segment's outcome with exactly one of:

- `lym loom circling continue pr-review-row` for a round over the amended record
- `lym loom circling accept pr-review-row` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/_board/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add pr-review-row --by parent --title … --decision … --rationale …`, run in `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept pr-review-row` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue pr-review-row` runs one more round.
3. Resume the run with `lym loom resume` in `/home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/pr-review-row`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
