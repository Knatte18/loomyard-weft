# Card 9 — checklist-block-marks

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistVisitsOf` -> `func checklistVisitsOf(status shedengine.Status, blockRows map[string]bool, startedAt string, paused []pauserecord.Interval, now time.Time) (visits map[string]checklistVisits, order []string, history []checklistVisit)`
- `internal/loomcli#checklistVisit`
- `internal/loomcli#checklistVisitLine`
- `internal/loomcli#checklistReviewBlock`
- `internal/loomcli/doc.go`
- `internal/loomcli#TestRenderChecklist`
- `internal/shedverbs#RenderStatusHuman`
- `internal/shedverbs#TestRenderStatusHuman`

**Create:**
- `internal/loomcli#checklistEnteredWord`
- `internal/loomcli#checklistBlockName`
- `internal/shedverbs#blockAwareLast`

**Uses:**
- `internal/loomrecipe#RecipeRow`
- `internal/shedverbs/doc.go`

**Intent:** Make `lym loom status` show a block (D3, D6), both on its terminal view and on its checklist pane: a block row's Done reads as entered, and an inlined row's block stands beside its name.

The checklist pane, which `lym loom start` keeps drawing from the status file:
- `checklistVisitsOf` takes the set of block rows, the rows of `in.rows` whose `IsBlock` is true, which `renderChecklist` builds; a Done entry of a block row reads as `checklistEnteredWord`.
  With the recipe unreadable the set is empty and the entry reads `done` as stored.
- `checklistVisit` gains `block`, the entry's `HistoryEntry.Block`.
- `checklistBlockName` renders `<name> in <block>`, and `<name>` alone for an empty block; a row line of a row whose `RecipeRow.BlockPath` is set renders its name through it as `<n>. <name> in <block path>`.
- `checklistVisitLine` writes a non-empty row through `checklistBlockName` with the visit's block, so the history-order list of an unreadable recipe and a review block's visit lines name the block beside the row.
- `checklistReviewBlock`'s segment line names the block of the segment's rows the same way.
- `doc.go`'s checklist paragraph, beside the `bounced` and `escalated` sentence, says a block row's Done visit reads `entered` and an inlined row is named `<row> in <block path>`.
A status file without `block` reads as before.

The terminal view, `RenderStatusHuman`, which `lym loom status` prints:
- its `now` line names the current row's block from its routing definition's `Block`, as `now  <row> in <block>`;
- its `last` line comes from `blockAwareLast`: when the latest history entry is a Done of a row whose routing definition carries a skip target, a block row, it reads `<row> → entered`; when the latest entry carries a block, its row reads `<row> in <block>`; otherwise it is `Activity.Last` as today.
A routing with no producers, and a run whose latest entry is outside every block, print as before.

Tests, tier0:
- `TestRenderChecklist` gains rows over rows shaped like arras's expanded `Rows` and a history carrying `block`:
  - the `Motif` row's Done visit reads `entered`;
  - `Discussion-Write`'s row line reads `3. Discussion-Write in Motif`;
  - the Discussion-Review block's segment line names `Motif`, and its visit lines name each row with `in Motif`;
  - with the recipe unreadable, a visit line of an inlined row names the row `in Motif`, and a status written without the field renders unchanged.
- `TestRenderStatusHuman` gains rows over a routing with a block row (`SkipTo` set) and inlined rows (`Block` set): a latest Done of the block row prints `last Motif → entered`; a run at an inlined row prints `now  Discussion-Write in Motif` and its latest entry's `last` line names the row `in Motif`.

**ImpactSummary:** Rendering only, in loomcli's checklist and shedverbs' terminal status view; `checklistVisitsOf` has the one caller `renderChecklist`, and no output changes for a row outside a block.
