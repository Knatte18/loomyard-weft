---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `config-keys-nested-by-area` has stopped at round 1 and is `awaiting` a decision.
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

- `F4` (MEDIUM): The board entry asks that run_settings, profiles and `lym config --set` use the dotted full name, file included; the record reads that as the existing nested module object and `lym config loom --set discussion.review_ask_parent`, and puts the literal one-token form in Scope Out and Rejected alternatives. Whether the entry wants a one-token form is a question about what the task is for, which only the parent settles. Fix: Parent decision: confirm that the nested run_settings object and `lym config <module> --set <dotted path>` satisfy the entry's run_settings and --set bullet, or ask for a one-token form. The writer then drops the Open risks line and records the answer, or adds the form as a Decision.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D4 gives every refusal the one way forward `lym config reconcile`, which moves it, but that command moves only a worktree's own and local files; it does not reach a hub base file (needs --layer default), a profile block (moved only by D6's once-per-hub reinterpretation, which no reconcile verb reruns), a task's run_settings (moved only by D7's walk step), `--set` or a recipe reference (fixed only by the new name); Seed-Child also wraps a run_settings refusal in its own `lym board upsert` way forward, so the operator gets two. This breaks PATTERN-refusal-way-forward. Fix: In D4, replace the single way forward with one per site: a local or worktree file names `lym config reconcile`; a hub base file names `lym config reconcile --layer default` from the prime; a task's run_settings names `lym board upsert` with the new full name, and say Seed-Child's own wrapper carries that same text so only one way forward is printed; a profile block names editing profiles.yaml with `lym config profiles` to the new path; `lym config <module> --set` and a recipe setting reference name the new full name. Update the D4 acceptance criterion at line 186 to assert the per-site way forward, and the plan-writer note at line 209 and D5's bound at line 111 to match.
- `F2` (BLOCKING): D6 runs after configsync's FollowTemplate merge of profiles.yaml, so a shipped profile whose loom.code_max_bounces the operator edited (1 to 3) keeps code_max_bounces 3 as the operator's unit and also gains the template's code.max_bounces 1; D6's move then collides, and the shared configreg helper (D7) under D5's new-path-wins rule drops the operator's 3 for the template default, against the Goal that every existing value carries over. Fix: In D6, state the collision rule for profiles.yaml: either move the old keys of the file and of its template base (profiles.template-base) before the follow, inside configsync, so the follow sees the operator's value at the new path; or, when the reinterpretation finds both, keep the old key's value whenever it differs from the template base's value at the old key. Name which of the two in D6 and say the D7 helper takes that rule as a parameter if it differs from D5's. Add an acceptance criterion: a seeded profiles.yaml whose shipped solid.tapestry block the operator changed to loom code_max_bounces 3 holds loom code.max_bounces 3 after the walk.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, except that over a CONTINUE or CIRCLING verdict an open BLOCKING ledger entry of another class holds it: the round then runs another round, or inside the soft-cap window escalates again with cause budget, which only the parent accepts. An `accept` at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add config-keys-nested-by-area`, then record the segment's outcome with exactly one of:

- `lym loom circling continue config-keys-nested-by-area` for a round over the amended record
- `lym loom circling accept config-keys-nested-by-area` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area`.

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

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add config-keys-nested-by-area --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept config-keys-nested-by-area` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue config-keys-nested-by-area` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/config-keys-nested-by-area`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
