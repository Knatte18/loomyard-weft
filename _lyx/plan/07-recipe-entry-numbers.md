# Card 7 — recipe-entry-numbers

**Create:**
- `internal/loomrecipe#EntryNumbers`
- `internal/loomrecipe#TestEntryNumbers`

**Edit:**
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe/doc.go`

**Intent:**
One recipe-rows lookup covers every embedded recipe, batten's included (D3).
`Rows` (and `RowNames` through it) serves `shedrun.RecipeBatten` from `recipes.BattenRecipe` beside the loom and darn recipes; any other name is still refused with an error naming it.
Both functions' doc comments, which list the recipes each name selects, name the batten recipe too.
A recipe added to this lookup is numbered with no change to its readers.

`EntryNumbers`, in `names.go`, numbers a recipe's entries `1`, `2`, … in row order, the numbers every pane draws.
A review segment, a non-empty segment holding a Bouncer row, is one entry: it takes one number, at the place of its first row, and every row of the segment maps to that number.
Every other row, a row of a segment with no Bouncer row such as `PR-Review` included, is an entry of its own.
A row that is not in `rows` has no key in the result, which is how a reader tells a history-only row.

Tests:
- `TestRowNames_FollowEachRecipesRowOrder` gains the batten recipe in its table, and its refusal loop asks for a name no recipe has instead of `shedrun.RecipeBatten`; its doc comment, which says a recipe the loom module does not own is refused, says an unknown recipe name is.
- `TestEntryNumbers` pins the numbering on the shipped recipes and on a constructed one: in the loom recipe `Preflight` is 1, `Discussion-Write` 3, all three `Discussion-Review` rows 4, `Plan-Write` 5, all three `Plan-Review` rows 6, and `PR-Gate` 15 and `PR-Rework` 16, a segment with no Bouncer row numbering each row; in the batten recipe `Run-Shed` is 6 and `Land` 7; in the darn recipe the `Darn-Review` rows share one number; a constructed recipe whose segment has no Bouncer numbers its rows apart, a name not among its rows has no key in the result, and an empty row list gives an empty map.
  The package's row in `cmd/lyx/testdata/test-budget.yaml` budgets one test more than the package has today, so this test fits it.

`doc.go` names `EntryNumbers` and the batten recipe in its `Rows` passage.

**ImpactSummary:** `Rows` admits one more recipe name, which its two callers (`loomcli`'s checklist and peers) never pass today; `EntryNumbers` is new and unused until cards 9 and 12.

**Commit:** 7: loomrecipe: rows of every recipe and their entry numbers
