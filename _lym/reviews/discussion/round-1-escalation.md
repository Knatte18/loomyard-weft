---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `fabric-single-repo-mode` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F16` (MEDIUM): The board entry says the operator wants this mode taken in if fabricengine is rewritten (note clean-room-module-rewrite), while the record builds it into the current package now; whether to build now or only spec it for the rewrite is open for the parent (support log Q6) Fix: The parent decides whether the run builds the mode into the current `internal/fabricengine` now or writes only a spec for the rewrite, and records the call with `lym loom decision add`.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D8 publishes a view before every open, update and merge but never says the publish is idempotent, so each publish makes a new view SHA, which resets GitHub's required checks and stale approvals right before Finalize merges, and a resume repeats the cycle Fix: In D8, state that the publish reuses the existing `lym-view/<branch>` tip when its tree and its parent equal the ones it would write, and otherwise writes a commit with fixed author and committer identity and dates so the same inputs give the same SHA. Add to the D8 acceptance criterion that two publishes of an unchanged branch leave the view ref on the same SHA and push nothing the second time.
- `F2` (BLOCKING): Unpushed local records commits make every parent read as ahead of its upstream, so Finalize's parent alignment halts on ParentAhead or ParentDiverged and FastForwardPair refuses as not a fast-forward, which breaks D5's claim that a fast-forward to origin is unchanged and D8's fast-forward of the parent after a GitHub merge Fix: In D5, replace the sentence that a fast-forward to the branch's own origin tip is unchanged with the rule that bringing a branch level with its origin tip first commits the worktree's records, then replays the local records-only commits onto origin's tip as D7 replays them, and refuses only when a local non-records commit is unpushed. In D8, say that after GitHub merges the view Finalize brings the local parent level by that rule instead of a pure fast-forward. In D9, add the local-versus-upstream tip relation (`fabricengine.Fabric.ReadParentTips`, Finalize's parent alignment) as a reader kind that judges code commits only. Add an acceptance row: with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete.
- `F3` (BLOCKING): D7 and D2 push records to the prime's branch, the operator's code branch, with no bound for a branch that refuses direct pushes, so every prime records push is owed for good, every run held at a prime transition waits for ever, the named way forward git push is refused too, and --single-repo fails half-done Fix: Add to D7 a decision for a prime branch whose origin refuses a direct push (lym already recognises the refusal through `protectedBranchRefusal` in `internal/landingshed/finalize.go`): either `--single-repo` and the one-URL clone refuse such a hub at clone time, naming the protection as the cause, or the prime's records go somewhere a push is accepted. State which, and the way forward the refusal names. Add a D2 bound and an acceptance row for a protected prime branch.
- `F4` (BLOCKING): D7's bound that fabric never rewrites a code commit, and its prime push guard, rest on three predicates the record never defines (a records-only commit fabric made, a commit carrying fabric's merge trailer, a commit origin lacks), and fabric reads trailers from any message line, so a forged trailer can pass the replay or the guard Fix: In D7, define each predicate as the git read that decides it. A records-only commit is a non-merge commit whose changed paths all lie under the records paths of both its parent and itself, whatever trailer it carries. A commit carries the merge trailer only when it is a merge commit fabric wrote. Pick and state either that a merge's second-parent commits count as lacking or that lacking means not reachable from any origin ref. State the bound: a commit message alone never admits a commit that changes code paths.
- `F5` (BLOCKING): D6 keeps the hand-over as today, but the owed-code-tip paths (pushOwedCodeTip, deleteGivenUpCodeBranch, ReturnToPreviousTransition) act on the code branch separately from the records branch; with one branch they rewind origin off the transition records or delete the branch that holds them Fix: In D6, state that in a one-repository hub no code tip can be owed apart from its records, because one push carries both. So the owed-code-tip push, the given-up code branch delete and the return to a previous transition do not run there. A rejected push or a takeover goes through D7's replay and today's hand-over only. Add an acceptance row: after a takeover in a one-repository hub, origin's branch keeps every transition commit.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add fabric-single-repo-mode`, then record the segment's outcome with exactly one of:

- `lym loom circling continue fabric-single-repo-mode` for a round over the amended record
- `lym loom circling accept fabric-single-repo-mode` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add fabric-single-repo-mode --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept fabric-single-repo-mode` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue fabric-single-repo-mode` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/fabric-single-repo-mode`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
