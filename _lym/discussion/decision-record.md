## Goal

```markdown
Imported from [issue #155](https://github.com/Knatte18/loomyard/issues/155).

## What was seen

In a darn run's PR-Review row, `lym loom pr-findings` failed twice on a findings file that passes the burler format check, with: `loomrecipe: shedbuild: producer 13 "Finalize": landingshed: NewFinalize: Deps.OpenFabric must not be nil`.

## Workaround

The PR-Review agent fixes the parent's findings by hand and commits, without `pr-findings`.

## Expected

`pr-findings` builds the recipe's shed with its fabric dependency set and runs the findings round.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - regression-test-bypasses-verb (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
