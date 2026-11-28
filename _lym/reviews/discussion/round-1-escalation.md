---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `recipe-build-from-sketch` has stopped at round 1 and is `awaiting` a decision.
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

- `F8` (MEDIUM): The board note `plan-and-build-recipes`, filed after this Discussion in the same project `plan-ahead-chains`, splits tapestry's post-Discussion rows into a plan recipe and a build recipe. That conflicts with D1's one jacquard recipe holding them all Fix: For the parent: either keep D1 as the entry asks, with jacquard as one recipe of every row after Discussion and the note as later work that may split it, or redesign jacquard now to fit the note's plan/build split. Record the choice with `lym loom decision add`; the writer cannot settle it.
- `F9` (LOW): The board entry asks the task to add the 'jacq' nickname; D11 and the Open risks line hand that write to the parent after landing, and no board note answers it Fix: For the parent: accept D11, so the parent runs `lym heddle set` for the `nicknames` entry once the run lands, and record that with `lym loom decision add`. Or rule that a card of this run writes the nickname through `lym heddle set`, which changes D11 and drops the Open risks line.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D5's way forward for the `lym loom start` refusal, a board upsert to tapestry, cannot clear it: the start reads its recipe from the worktree's seed, which stays jacquard Fix: In D5, give each judging site its own way forward. Batten's start guard keeps `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`. The `lym loom start` refusal names D4's jacquard sequence: from the hub's prime worktree `lym batten cancel <slug> --remove`, `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, `lym board set-status '{"slug":"<slug>","status":null}'`, then `lym batten run <slug>`. Drop the clause saying a start with no status file can take the upsert, or confine it to batten's site. Change the D5 acceptance criterion from 'both messages carry the way forward' to: each site's message carries its own way forward, and the test reaches both.
- `F2` (BLOCKING): D4 sends a jacquard run down the RecordWrite path of the decision verbs, whose refusals name `lym loom goto --to Record-Write` and whose pending test reads Record-Write by name; jacquard has no such row Fix: In D4, after 'The decision verbs treat RecordSeed as they treat RecordWrite', add this. The RecordWrite-path refusals in `internal/loomcli/decision.go` (`decisionWayRecordStart`, `decisionWayRecordResume`, `decisionWayRecordGoto`, `decisionWayUncommittedGoto`) and `recordWritePending` name the row of the run's recipe that runs the record or seed engine, read from the recipe's rows through `loomrecipe.EngineRow`, never the literal Record-Write. Add an acceptance criterion: on a jacquard run past Record-Seed with an uncommitted record, `lym loom decision add` refuses with a way forward naming `lym loom goto --to Record-Seed` (tier 1).

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, except that over a CONTINUE or CIRCLING verdict an open BLOCKING ledger entry of another class holds it: the round then runs another round, or inside the soft-cap window escalates again with cause budget, which only the parent accepts. An `accept` at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add recipe-build-from-sketch`, then record the segment's outcome with exactly one of:

- `lym loom circling continue recipe-build-from-sketch` for a round over the amended record
- `lym loom circling accept recipe-build-from-sketch` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add recipe-build-from-sketch --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
   When the verb refuses because the task's scope is closed, name the applied decision and its source in your report back instead,
   and handle each decision finding as the cause detail says for a closed scope.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept recipe-build-from-sketch` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue recipe-build-from-sketch` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/recipe-build-from-sketch`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
