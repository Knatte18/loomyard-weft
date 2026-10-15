# Card 11 — checklist-gate-rework

**Create:**
- `internal/loomcli#checklistReworkSet`
- `internal/loomcli#checklistRework`

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistVisitLine` -> `func checklistVisitLine(visit checklistVisit, row string, reworkFor string) string`
- `internal/loomcli#checklistReviewBlock`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/doc.go`

**Intent:**
The checklist attributes the rework a gate row's refusal caused (D7).
A gate refusal is a visit that ended bounced, a Stuck entry with `routed_to`, of a row in no review segment (a segment holding a Bouncer row).
`checklistRework` walks the history-order visits once and returns a `checklistReworkSet`: the gate row each rework visit was rework for, keyed by the visit's history index (-1 for the open visit), and each gate row's rework total with whether it includes the open visit.
Its `perched` argument is the set of rows in a review segment, which `renderChecklist` builds from the recipe rows it already reads; a bounce by a row in that set is not attributed, since the review rounds are that rework and D2's round totals show it.
A refusal's rework visit is the history entry right after the bounce entry when that entry is a visit of the routed-to row; a goto entry or a visit of another row there leaves the refusal with no rework visit.
When the bounce is the last history entry and the open visit is of the routed-to row, the open visit is the refusal's rework visit.
`PR-Gate` and `PR-Rework` sit in `PR-Review`, which holds no Bouncer row, so a rejection's rework is attributed.

`checklistVisitLine` takes the gate row a visit was rework for, empty for none, and appends ` rework for <gate row>`.
Its callers are `renderChecklist`, whose per-row lists pass the set's entry for the visit's index and whose recipe-unreadable list passes empty, and `checklistReviewBlock`, which passes empty, since no visit inside a review segment is attributed this way.
`renderChecklist`'s row line of a gate row appends ` rework <time>`, the sum of its rework visits, drawn with `statuspane.FormatSeconds` when every rework visit is closed and with `statuspane.FormatMinutes` when the sum includes the open visit (D1).
With the recipe unreadable the frame cannot tell a review segment's bounce from a gate row's, so the recipe-unreadable list attributes no rework, rather than marking a Bouncer's bounces D7 excludes; `renderChecklist` builds no rework set there and passes empty to `checklistVisitLine`.

`TestRenderChecklist` gains rows over `reviewRecipe`'s `PR` segment:
- a `PR-Review` bounce routed to `PR-Rework` followed by a `PR-Rework` visit marks that visit ` rework for PR-Review` and adds its time to `PR-Review`'s line as ` rework <seconds>`;
- a bounce that is the last entry, with `PR-Rework` the open visit, marks the open visit and draws the gate row's rework in whole minutes;
- a bounce followed by a goto, and one followed by a visit of another row, mark nothing and add no rework part;
- a bounce that is the last entry while the open visit is of another row, and one that is the last entry of a done run, mark nothing;
- two `PR-Review` refusals, each followed by a `PR-Rework` visit, mark both visits and add their summed time to `PR-Review`'s line;
- a `Rev-Bouncer` bounce inside the review segment marks nothing;
- with the recipe unreadable, a `PR-Review` bounce followed by a `PR-Rework` visit marks nothing in the history-order list.

The package doc's checklist sentence names the rework attribution.

**ImpactSummary:** frame text in `internal/loomcli/checklist.go` and one private signature whose two callers this card edits; no record changes, since the status history already records each route.

**Commit:** 11: loom checklist: rework after a gate row's refusal
