# Round 3 fixer report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed.
  Premise checked: `goPackageDir` in `internal/planparser/targetdirs.go` maps quarry's `<dir>_test` pseudo-unit to its package directory, and `PATTERN-glyph-conversion-chokepoint` sanctions that mapping.
  D4's unit sentence now reads "through `Glyph.UnitPath`, and for an external test package through planparser's one external-test unit mapping"; a sentence after the package-unit rule gives the external-test shape; the Bound and the `PATTERN-glyph-conversion-chokepoint` Constraints line name the mapping; a new acceptance criterion covers an import-only change in a `_test.go` file of package `<dir>_test`.
- F2 (LOW), fixed by the review's first option.
  Premise checked: `contracts/recipes/darn-recipe.yaml` shares Preflight, Describe, Merge-In, Publish, PR-Gate, Finalize and Friction-Reflect with loom and adds Darn-only rows.
  D5 now states that every peer is read against the loom recipe, a darn run's shared row names take their loom positions and its darn-only rows read 0, an unknown order.
  The criterion on the embedded-recipe fields gains the darn cases (Publish gets loom's position, Darn-Write gets 0).
  The loom-recipe option was taken over per-run recipes because it needs no recipe name in the run status and the verdict is unchanged either way.
- F3 (LOW), fixed by the review's first option.
  D5's parent-commit test also escalates a commit that already makes one of this run's decisions, matching the peer rule's duplicate case and Q11; the parent-commit criterion is extended to match.
- Support log: a Round 3 entry lists the three fixes, and Q11 records the parent-commit duplicate case.

## Disputed

None.

## Deferred

None.
