# Card 2 — loomrecipe-rows

**Create:**
- `plan:internal/loomrecipe#RecipeRow` -> `type RecipeRow struct`
- `plan:internal/loomrecipe#Rows` -> `func Rows(recipe string) ([]RecipeRow, error)`

**Edit:**
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`

**Intent:** The checklist (D1, D2) needs each recipe row's name, its `segment` and whether it is a Bouncer row, so it can tell a review segment (a segment holding a Bouncer row) from a plain one such as `PR-Review`.
Add `RecipeRow` in `internal/loomrecipe/names.go` with the fields `Name`, `Segment`, `OnStuck` and `Bouncer` (true when the row's engine is `bouncerEngine`), and `Rows`, which parses the named recipe exactly as `RowNames` does today (`shedrun.RecipeLoom` or `shedrun.RecipeDarn`, any other name refused naming it) and returns its rows in file order.
`RowNames` becomes the names of `Rows`' answer, so the two never disagree; the file header names both.
The checklist reads `Rows` so it needs no `shedbuild` import of its own.

`TestRowNames_FollowEachRecipesRowOrder` extends to `Rows`: for each recipe, the names equal `RowNames`' answer, each row's `Segment` and `OnStuck` equal the parsed row's `segment` and `on_stuck`, and `Bouncer` is true exactly on the rows whose engine is `Bouncer`; the unknown-recipe refusal is asserted for `Rows` too.

**ImpactSummary:** RowNames keeps its answer and its callers (checklist, peers); one new exported reader in loomrecipe.
