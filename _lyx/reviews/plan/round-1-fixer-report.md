# Fixer report — plan round 1

## Changes

- **F1** (BLOCKING), fixed as proposed in `_lyx/plan/01-replace-stencil-claims.md`.
  Premise confirmed: `grep -c specs_dir` over `contracts/stencils/loom/loom-rubric-*.md` gives darn-review 0 and every other rubric at least 1.
  The `TestLoomRubrics` paragraph now gives the table a per-row `namesSpecs` boolean, true for the discussion-, plan-, webster- and merge-review rubrics and false for darn-review, and asserts the rendered text contains `/abs/specs/dir` exactly when the flag is set, in the same equality shape as the `publish_failure`, `webster_record` and `merge_review` checks.
  It states that the darn-review rubric carries no `{{.specs_dir}}` today and that card 8 adds none.
- **F2** (LOW), fixed as proposed in `_lyx/plan/01-replace-stencil-claims.md`.
  Premise confirmed: the `//testtiming:keep` line sits on `TestStencils_TopLevelMarkers` at `contracts/stencils/markers_test.go:15`, and `rubric_test.go` has none.
  The sentence moved out of the `TestLoomRubrics` paragraph to the end of the `TestStencils_TopLevelMarkers` paragraph and names that test outright; the quoted replacement reason is unchanged.

## Disputed

None.

## Deferred

None.
