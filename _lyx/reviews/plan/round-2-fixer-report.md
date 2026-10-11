# Plan-Review round 2 — fixer report

Every finding was checked against the code and the decision record, and every one held.
All five are fixed.

## Fixed

- **F1 (MEDIUM, consistency)**: card 6 now lists `contracts/specs/shed-recipe-spec.md` and `contracts/recipes/loom-recipe.yaml` under `**Edit:**`.
  I confirmed the three stale statements at `contracts/specs/shed-recipe-spec.md:62`, `internal/shedrecipe/recipe.go:61` and `contracts/recipes/loom-recipe.yaml:309-314`.
  The Intent now says how each one is rewritten.
  The spec sentence and the `VerifyMergeBase` field comment now say that `VerifyWholeDiff` alone selects the form, and that a set reader is the base of the headroom lint, or of both lints under the whole-diff form.
  The recipe's `gates` comment now names the headroom lint ahead of the comment lint.
  Both files exist, so `path-missing` admits them.
- **F2 (LOW, design)**: card 10's ceiling is now split.
  A trimmed line that already carries a link ends at most 200 bytes, and each of the six lines that gains a link ends at most 230 bytes, link included.
  The Intent also says a clause of the requirement itself never leaves the line, and names `PATTERN-push-both-sides` as the case that rule protects.
  I recomputed the margin over the current `PATTERN.md` (16217 bytes): the 27 linked lines free 1223 bytes and the six unlinked lines free 91, 1314 in total against the 1217 needed.
- **F3 (LOW, scope)**: card 3 no longer renames `TestLintCommentsCmd_OverARepository`.
  The test keeps its name, so its Edit glyph stays valid, and only its doc comment and the file header of `lintcomments_integration_test.go` change to name both verbs.
- **F4 (LOW, consistency)**: card 2's `**ImpactSummary:**` now says that `Check`'s oversize message gains the bytes over the cap.
  It names the message's readers: `TestCheck`, the failure output of `TestOverview_RealFilePassesCheck` and, through that output, Publish's failure record.
  It also states that no code parses the message.
- **F5 (NIT, consistency)**: card 1 now says it runs a four-row table, which matches the four outcomes it lists.

## Disputed

None.

## Deferred

None.
