# Fixer report — plan, round 1

## Changes

- F1 (fixed as proposed): card 3's Intent gains a bullet for the stencil's caller rule in `contracts/stencils/loom/loom-template-plan.md`.
  For a caller in a `_test.go` file the card lists only the member glyph whose body holds the call, matching card 2's `caller-uncovered` way forward.
  The bullet keeps "lists every caller of it" and "on the same or an earlier card for a delete" so the existing `wantAll` claim in `contracts/stencils/claims_test.go:295` still passes.
- F2 (fixed as proposed): card 1's `TestGateParity_ReworkPlanGate` bullet now has `reworkParityFixture` set the card's `ImpactSummary` when `edit` is set, so the row's only blocking finding is `edit-test-file`.
  `plankit.Card` already has an `ImpactSummary` field (`internal/testkit/plankit/plankit.go:52`).
- F3 (fixed as proposed): card 3 now quotes the rubric sentence with "too", as it reads in `contracts/stencils/loom/loom-rubric-plan-review.md:63`.

## Disputed

None.

## Deferred

None.
