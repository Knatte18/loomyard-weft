---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `hardener` has stopped at round 1 and is `awaiting` a decision.
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

- `F10` (MEDIUM): The Open risks treadle question stands - D8's rationale cites the 2026-10-16 decision as keeping treadle's stencils only until Hardener is built, but the entry says they are kept for Hardener and re-derived from the Bouncer judge when it is built, and the 2026-11-18 operator decision names treadle among the modules to build on; neither settles whether treadle may be deleted. Fix: Raise to the run's parent - may this run delete internal/treadleengine, its stencils and roles (D8), or must they stay? Once answered, the writer drops the Open risks line and restates D8's rationale to quote the board entry's note as written.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D8 rests on the false premise that internal/agentconfig refuses no unknown role; Load refuses a skills or efforts key naming a role outside the roles lym starts, so a hub agents.yaml naming treadle-judge stops loading once the role is deleted, and D8's acceptance criterion fails as designed. Fix: In D8, replace the sentence about agentconfig with the observed behavior and its pin (internal/agentconfig/config.go Load and skillLists; config_test.go rows UnknownRole and PairWithUnknownRole), and decide that a configreg.Reinterpretation on the agents module strips the treadle-judge and treadle-targeting keys from skills and from role/model efforts keys, applied by hubreconcile per PATTERN-config-layers. Rewrite the D8 acceptance criterion to - a hub agents.yaml naming treadle-judge is rewritten by the reinterpretation and then loads (D8). In support-log.md, correct the Turn 3 bound-breaker line that records the false fix.
- `F2` (BLOCKING): The live gate runs lym gate test --tier 2 and --tier 2 --copies over a package list, naming a tier instead of a gate key, against PATTERN-test-speed's declared text (via lym gate test by gate key) and its background rule that a site names a gate key, never a tier list. Fix: In D4, have the live gate run two hub gate.yaml keys shipped the way code-burler-fix is, for example harden-live (tier 2, the serial run) and harden-live-copies (tier 2, the concurrent run), through lym gate test --gate <key> with the packages as targets; make --copies go with --gate and state that bound. Replace --tier 2 in D4 steps 1 and 2, in the --copies bound and in the D4 acceptance criteria with the key form, and keep step 3's rule that no key names tier 3.
- `F3` (BLOCKING): D5's reaps kill the sandbox hubs' processes found by a /proc scan, but PATTERN-child-processes ends a run's processes only through runproc's End, EndSparing, EndRole or EndSubject, with EndProcess reserved for lym tender reap's orphans; D5 names no runproc route and declares no PATTERN change. Fix: In D5, have the reaps first end each sandbox hub's recorded processes through runproc (End over that hub's owners), and kill by the /proc scan only what no record names, stating that this second route is a new orphan reaper. Add the record's PATTERN changes section with a changes bullet for PATTERN-child-processes that admits the Hardener sandbox reaper beside lym tender reap as a user of EndProcess, bounded to processes under the sandbox root; add a matching acceptance criterion.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add hardener`, then record the segment's outcome with exactly one of:

- `lym loom circling continue hardener` for a round over the amended record
- `lym loom circling accept hardener` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/hardener/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/hardener/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/hardener/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/hardener/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/hardener`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add hardener --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/hardener`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept hardener` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue hardener` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/hardener`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
