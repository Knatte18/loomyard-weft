---
round: 2
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 2

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `gate-scope-by-coverage-map` has stopped at round 2 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F4` (MEDIUM): Open question for the parent - may D2 change testmap's staleness rule and raise its rebuild cost now, when note testmap-upkeep says it owns the whole staleness policy, dependency drift included Fix: Raise to the run's parent. Once the parent's answer is recorded with lym loom decision add, the writer either keeps D2's staleness change and drops the Open risks line, or removes D2's staleness rule and states in D5 that a dependency-drift miss lasts until testmap rebuilds the map by its present rules.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): A non-Go file in a covered package's subdirectory, such as an embedded stencil under contracts/stencils/loom/, is neither recorded nor compared, so its change narrows importers to nothing selected Fix: In D2, record per covered package every tracked file that impactset's path-to-package rule maps to that package, meaning every file under its directory outside a nested package directory, go:embed targets included, not only the files directly in the directory. In D5's last bullet, compare that same set. Add a D5 acceptance criterion where an edit to an embedded file in a subdirectory of covered package A runs importing package B whole with a miss reason.
- `F2` (BLOCKING): A linked in-module package with no covered block, such as a package of only constants or types like internal/lymdirs, which declares no function, is never compared, so changing its constant or type selects nothing Fix: In D2, record the file set and hashes of every in-module package the test binary links (the coverpkg list), not only the packages a test covered a block of. In D4 and D5, compare every package so recorded, so a non-body change in a linked package with no covered block is a D5 miss. Reword the D2 acceptance criterion so it no longer says not stale when only an uncovered package changed, and state the staleness rule for linked-but-uncovered packages explicitly. The cost question for that staleness goes with F4's parent question.
- `F3` (BLOCKING): The non-body miss, judged from the map's own Commit, fires on most landings to widely linked packages, so in common flows importing packages fall back to whole runs; the record leaves that hit rate unmeasured in Open risks Fix: Measure, from main's history over a recent window, how often a landing changes a widely linked package outside function bodies, and record the number with its source in Constraints. Then narrow D5's non-body rule so an added declaration that no unchanged code can reach is no miss, the way a plain function already is. That covers an added const, an added var without an initializer that runs code, and an added type with no methods. Keep as misses a changed or removed declaration, a struct field change, a var initializer and a method-set change. Add a D5 acceptance criterion for an added const in A being no miss. If the measured rate still leaves coverage close to diff once testmap's cadence is counted, name that dependency as a question for the parent in Open risks, tied to note testmap-upkeep, in place of the current third Open risks bullet.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add gate-scope-by-coverage-map`, then record the segment's outcome with exactly one of:

- `lym loom circling continue gate-scope-by-coverage-map` for a round over the amended record
- `lym loom circling accept gate-scope-by-coverage-map` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map/_lym/reviews/discussion/round-2-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map/_lym/reviews/discussion/round-2-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map/_lym/reviews/discussion/round-2-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add gate-scope-by-coverage-map --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept gate-scope-by-coverage-map` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue gate-scope-by-coverage-map` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/gate-scope-by-coverage-map`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
