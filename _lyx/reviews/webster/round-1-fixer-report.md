# Webster round 1 — fixer report

## Fixed

- **F1** (MEDIUM, consistency), `contracts/stencils/claims_test.go:249-251`: re-indented the three new chair-stencil claims rows from three tabs to two, matching their sibling rows.
  `gofmt -l contracts/stencils/` now lists nothing, and `lyx gate test ./contracts/stencils` passes.
  Commit `2befea4f4`.
  No test was added, since the finding is formatting, not a coverage gap.

## Disputed

None.

## Deferred

None.
