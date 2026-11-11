## Goal

```markdown
Imported from [issue #115](https://github.com/Knatte18/loomyard/issues/115).

## Seen

`lym beam merge-parent <beam>` names its merge task `<beam>-merge-in` every time.
A beam that already merged its parent once, through a completed and torn-down `<beam>-merge-in` run, gets a second task with the same slug.
Dobby then refuses to start it: `"<beam>-merge-in" has already completed; to run it again, run "lym batten cleanup --apply" from the prime ... or give the new task a different slug`.

## Workaround

The beam orch renamed the new task to `<beam>-merge-in-2` with `lym board rename`, and dobby started it.

## Expected

`merge-parent` picks a slug that no completed run or leftover task branch holds, such as a numbered suffix, so a second merge of the parent starts without a hand rename.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - hubland-heldslugs-wiring-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
