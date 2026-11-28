## Goal

```markdown
## Seen (`docs/research/webster/gate-time-2026-11-28.md`, lever A)

Since the card gate scope grew in `a69580709`, the batch gate takes 24.1 min per row against 6.6 before, mostly its tier1 run over the dependent packages.
That tier1 listed run takes 18.9 min per row.
None of the batch gate's tier1 failures after the change was in a dependent package, and the plan-level verify still runs tier1 over everything.

## Change

The batch gate's tier1 runs over the packages of the batch's own cards only, not their dependents.
Tier0 and the card gate keep their scope, and so does the plan-level verify.

## Expected

About 14.6 min saved per Webster or Weaver row; the report's estimate is 4.3 min for the narrower run against 18.9 min today.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - narrowed-tier-main-packages-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
