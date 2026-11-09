```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F2 deletions, F3 rows, F4 and F5 docs claims). No findings.

- **F2 deletions:** every assertion the reviewer said was duplicated is pinned elsewhere, so no coverage was lost.
  - Arras is in `TestRowNames_FollowEachRecipesRowOrder` (`recipe_test.go:240`) and `TestEntryNumbers` (`recipe_test.go:356`).
  - `HoldsSplitPerch(arras)` is in `recipekeys_test.go:50`.
  - `TestDiscussionApprovalRowsCarryApproveSeam` loops over `RecipeArras` (`approveseam_test.go:156`).
- **F5 docs claims:** `config.go:653-654` makes `split_reviewer_fan` and `split_fixer_fan` guard keys, so "closed to the local layer" is correct. The comment at `config.go:617` says `code_reviewer_fan` is the only reviewer fan open to the local layer. `split_fixer_model` and `split_judge` are `none`-category keys, which matches the doc text.
- **F5 null and 0 fallbacks:** the sentence about `split_max_bounces` and `split_hard_max_bounces` taking the shared values matches the existing per-perch wording at `docs/overview.md:1343-1346`.
- **F3 rows:** all pass in the gate. The `ResolveJudge` rows use a substring match, which is weak, but it matches the existing rows, and the `opus`, `haiku` and `sonnet` expectations differ, so the rows still discriminate.
- **F1 test:** the history assertions distinguish clearing from not clearing, so the goto rows would fail if the perch behaved differently. The `round-1-review.md` stat in the settled-perch row also passes after a clear, since the fresh cycle rewrites that file. It adds nothing there, but the row is not wrong.
- **F4 docs:** no sibling list was missed in the lines I read. `docs/overview.md:1597` ("`tapestry`, `darn` or empty") is a different context, the Board task's recipe.

Tool calls made:
1. Grep of the `loomrecipe` tests for the test names and `RecipeArras` uses, to confirm the deleted assertions are covered elsewhere (my context did not hold those tests).
2. Read of `internal/loomengine/config.go:590-670`, to check the guard and local-layer claims in the F5 docs (my context did not hold the config spec).
