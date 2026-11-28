## Goal

```markdown
Imported from [issue #284](https://github.com/Knatte18/loomyard/issues/284).

## What happened

A session timing packages through `lym gate test` found that a call's wall time includes the wait for a gate slot,
up to 300 s in that run.
Timing the calls therefore did not measure the test runs, and the session took its wall figures from `wall_seconds` in each run record.

## Why it is a problem

A caller who measures a gate call from outside gets a figure dominated by contention for slots,
and has to know about the run records to get the run's own time.
The note does not say whether the call's own output separates the two.

## What to do

Check that the result of `lym gate test` shows the slot wait and the run's wall time as separate values.
If it does not, add them; if it does, document in the verb's help where to read the run time without the wait.

Provenance: task `test-time-measure`, note `Darn-Write.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - printed-wait-untested-on-real-paths (class design, severity MEDIUM, open in rounds 1)
  - rerun-line-hides-slot-wait (class design, severity MEDIUM, open in rounds 1)
  - summary-patterns-require-wait-field (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
