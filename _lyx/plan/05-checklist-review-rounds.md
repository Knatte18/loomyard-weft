# Card 5 — checklist-review-rounds

**Create:**
- `plan:internal/loomcli#checklistReviewBlock` -> `func checklistReviewBlock(segment string, rows []loomrecipe.RecipeRow, visits []checklistVisit, status shedengine.Status) []string`

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistVisit`
- `internal/loomcli#checklistVisitsOf`
- `internal/loomcli#checklistBox`
- `internal/loomcli#TestRenderChecklist`

**Uses:**
- `plan:internal/shedadapters#OutputRound`
- `plan:internal/loomrecipe#RecipeRow`

**Intent:** D2: a review segment renders as one block, grouped by round in run order.

A review segment is a recipe `segment` holding a row whose `RecipeRow.Bouncer` is true; a segment without one, such as `PR-Review`, keeps rendering its rows as card 4's per-row lines.
`renderChecklist` draws each review segment as one block at the place of its first row, through `checklistReviewBlock`, and skips the segment's other rows in the recipe loop.

`checklistVisitsOf` records, beside each row's visits, every visit in history order with its row, its history index and its outcome, and now also records a goto entry as a zero-time line (outcome `goto`) that counts as no visit and adds no time, as today; card 4's per-row lines keep leaving goto entries out.
`checklistVisit` gains the fields this needs: `index` (the history index, -1 for the open visit) and `isGoto`.

The block, top to bottom:
- A heading line: `<box> <segment> <time> rounds <n>`, where the box is the box of whichever of the segment's rows is current (through `checklistBox`), else done when any row was visited, else not reached; the time is the sum of its rows' visit times; n is the number of rounds opened. An unvisited segment is one bare `[ ] <segment>` line.
- Lines placed before any round opened, indented two spaces.
- One sub-heading `  round <n>` per opened round, in the order opened, each followed by its lines indented four spaces, in the order they ran, each naming its row through `checklistVisitLine`.
- A Bouncer `done` entry that ends the segment, as the block's last line, indented two spaces, outside any round.

Placing the segment's history lines, walking them in history order:
- A review-row entry (an entry of the segment's review row, the Bouncer row's `RecipeRow.OnStuck` target) belongs to the round its output names through `shedadapters.OutputRound`: it opens that round when no earlier entry has, and otherwise joins it, so a hand-on entry (BudgetExempt Stuck, no spawn) over an opened round adds no heading, and a restart's hand-on over an unopened round opens it.
- A Bouncer entry read `bounced` waits for the next round a review-row entry opens, and stands first in that round, so round n reads bouncer, review, fix.
- A fix-row entry (any other non-Bouncer row of the segment), and a review-row entry whose output names no round, join the round in progress.
- A Bouncer entry that routes nowhere (`escalated`, or a `stuck` halt) stands as the last line of the round in progress, or before round 1 when none opened.
- A reroute entry (`OutcomeReroute`) whose `RoutedTo` names a row of the segment is treated as a goto into that row: it stands as the last line of the round in progress, or before round 1, and the review visit after it opens the next round by its output, with no Bouncer line.
- A goto entry naming a row of the segment stands as the last line of the round in progress, or before round 1 when none opened; the review visit after it opens the next round by its output, with no Bouncer line.
- The open visit of a segment row joins the round a waiting bounced Bouncer line opens (the round after the last), or else the round in progress, or else stands before round 1.

The checklist reads round numbers only through `shedadapters.OutputRound` (card 3), never by a parse of its own.
With the recipe unreadable, card 4's history-order list stands unchanged.

The review-segment rows of `TestRenderChecklist` build their recipe as `loomrecipe.RecipeRow` values holding a review segment (a Bouncer, a review and a fix row) and a `PR-Review`-like segment with no Bouncer row, and their outputs as `round-<n>-review.md` and `round-<n>-fixer-report.md` paths.
New rows of `TestRenderChecklist`:
- bouncer, review, fix, bouncer, review, fix, bouncer-done renders round 1 (bouncer, review, fix), round 2 (bouncer, review, fix) and a closing bouncer line.
- A hand-on review entry naming an opened round joins it with no new heading; a hand-on naming an unopened round opens it.
- A goto into the review row after round 1 renders the goto line at the end of round 1, then round 2 opening with its review line and no Bouncer line.
- A reroute into the review row after round 1 renders the same shape as the goto row.
- A goto into the segment before any round opened stands before round 1.
- A Bouncer that bounced while its review row is still running renders the Bouncer line and the open review visit under the next round's heading.
- A Bouncer awaiting line renders `escalated` as the last line of its round.
- A segment without a Bouncer row renders its rows as per-row lines.
- A run part-way through the review segment shows rounds 1..n, the open visit in the last round, and every later row a bare `[ ]` line.

**ImpactSummary:** Confined to the checklist file and its test; the frame of runs with no review segment is unchanged from card 4.
