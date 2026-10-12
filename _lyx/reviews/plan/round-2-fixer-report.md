# Round 2 fixer-report — plan

## Changes

- F1 (LOW, consistency), fixed as proposed.
  The premise holds: `contracts/stencils/claims_test.go:87` declares `type stencilClaims struct`, and the claim rows are entries of `var wordingClaims = []stencilClaims{` at line 107.
  - `_lyx/plan/04-twill-host-values.md`: the Edit entry `contracts/stencils#stencilClaims` became `contracts/stencils#wordingClaims`, and the Intent bullet on the `twill-directive.md` entry now starts with `wordingClaims'`.
  - `_lyx/plan/09-plan-twill-directive.md`: the same Edit entry change, and the Intent bullet on the `loom-template-plan.md` entry now starts with `wordingClaims'`.

## Disputed

None.

## Deferred

None.
