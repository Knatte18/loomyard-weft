# Card 9 — checklist-entry-numbers

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistReviewBlock` -> `func checklistReviewBlock(segment string, number int, rows []loomrecipe.RecipeRow, visits []checklistVisit, status shedengine.Status, arbiters *checklistArbiterPlacement) []string`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomrecipe#EntryNumbers`

**Intent:**
The checklist numbers its entries in recipe order, after the box (D3): `[x] 3. Discussion-Write 4m12s`.
`renderChecklist` takes the numbers from `loomrecipe.EntryNumbers(in.rows)`, so the frame of a run numbers by the rows its seed's recipe gives, a darn run by the darn recipe, with no recipe named in the checklist.
A review segment's block takes its segment's one number, which `checklistReviewBlock` now receives and draws after the heading's box (`[x] 4. Discussion-Review 21m00s rounds 2`); an unvisited segment's single line carries it too (`[ ] 4. Discussion-Review`); its rounds and their lines carry none.
A history-only row (`(not in recipe)`) has no number in the map and draws none, and the `recipe unreadable` fallback draws no number anywhere, since it has no rows.
The re-signed `checklistReviewBlock` has one caller, `renderChecklist`.

`TestRenderChecklist`'s wanted frames gain the numbers: plain rows, the review block's heading and the unvisited segment line numbered, the history-only row and every line of the recipe-unreadable rows unnumbered.
A new row renders a darn run's frame over `loomrecipe.Rows(shedrun.RecipeDarn)` and pins its numbers: `Darn-Write` second, the `Darn-Review` block third.

The package doc's checklist sentence says each entry is numbered and a review segment takes one number.

**ImpactSummary:** frame text in `internal/loomcli/checklist.go` and one private signature with one caller; the numbers come from `loomrecipe`, so a new recipe needs no checklist change.

**Commit:** 9: loom checklist: entry numbers fixed by the recipe
