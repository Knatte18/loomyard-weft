# Bouncer settles a spent review budget; escalate only stuck runs

At the soft cap (`review_max_bounces` or the perch's `<perch>_max_bounces`), the Bouncer now settles the round itself instead of escalating to the arbiter.
The arbiter was a cold agent spawn, and every recorded arbiter attempt on a budget escalation came back `continue` or `accept`.
That call is a classification of the findings, so the judge now makes it and Go applies it.

## What changes

- The judge tags every open ledger entry `fix: mechanical` or `fix: design-wrong`, from the review text alone.
  A missing or unknown tag reads as `design-wrong`.
- From the soft cap on, for every Bouncer row (loom's Discussion, Plan, Webster and Merge perches, and darn's Darn-Review perch), a `CONTINUE` or `CIRCLING` round with no decision is settled by a rule in `internal/shedadapters/bouncer.go`.
  An entry counts only when it is open, tagged `design-wrong`, and BLOCKING or MEDIUM.
  - None counts: accept unconverged through the existing accept path (carry-over, approve, commit), recorded as an automatic accept.
  - One counts at or past the hard cap: escalate with cause `hard-cap`.
  - An earned `CIRCLING`, or a key that counted in the previous round after a granted continue: escalate with cause `circling`.
  - Otherwise: record an automatic `continue` (cause `budget`) and run one more round, exempt from Shed's bounce-budget block.
  - Unreadable inputs (a parse-error row, an unreadable ledger): escalate with cause `budget`.
- New hard cap, never passed: `review_hard_max_bounces` (default 4) with per-perch `discussion_`, `plan_`, `webster_` and `merge_hard_max_bounces` in loom config, and `review_hard_max_bounces` in darn config.
  The loom keys are settable through `run_settings`.
  Config load refuses a hard cap below 1, a negative per-perch value, and a hard cap below the resolved soft cap, each naming the layer-specific way forward.
- `hard-cap` is a third escalation cause through the escalation record, decision file, parent notice and brief.
  A recorded `continue` at or past the hard cap is moved aside, with the round's escalation record, and the round re-escalates as `hard-cap`.
  Such a round's Awaiting Reason offers only `lyx loom circling accept`.
- `lyx loom circling continue` refuses at or past the hard cap, for every caller.
- The arbiter stencil's review rule is now one rule per cause, and the orch stencil's guidance follows the hard cap.
  The small-task advice becomes `{"loom": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`.
- Docs (`shedadapters`, `loomcli` and `darnengine` `doc.go`, `docs/overview.md`), the config template comments and `contracts/specs/refusal-spec.md` are updated.

## Design decisions

- The hard cap is measured by the round's number in the segment's current generation, not by Shed's bounce count, so a `lyx loom goto` that resets the count cannot let a round pass it.
- The rule's window opens at the soft cap or at a round numbered at or past the hard cap, whichever comes first.
- `*_auto_accept` keeps its meaning on a harvested round.
  Its replay path and the `blocking_escalates` path (Merge-Bouncer) now go to the rule instead of escalating.
- The arbiter's eligibility list is unchanged: `hard-cap`, `circling` and `budget` escalations still reach the arbiter and the Go driver.
- `lyx report convergence` and `report-timing` do not yet count rule-made decisions.

## To check

- A judge that mis-tags a design-wrong finding `mechanical` lets an unreviewed fix through.
  On Merge-Review and Darn-Review no later review reads the artifact, and PR-Gate lands it unread while `alignment_pr_approve` is true.
  Merge-Review carries open MEDIUM-or-worse entries into `## Open risks`; Darn-Review has no carry-over.
- A hub or task `run_settings` with a soft cap above 4 and no hard-cap key is refused at load, which parks a run in flight until a cap is changed.
- An older `lyx` binary, such as prod during a deploy window, refuses a `hard-cap` record it reads.
- A run already awaiting a `budget` escalation when the binary changes keeps that escalation.
- Not run by hand: the full rule against a live run with a real judge.
  The rule and the refusals are covered by unit tests.
