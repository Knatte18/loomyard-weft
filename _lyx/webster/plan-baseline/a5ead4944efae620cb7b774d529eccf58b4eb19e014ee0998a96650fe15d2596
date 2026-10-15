# Card 8 — checklist-seconds-round-totals

**Create:**
- `plan:internal/loomcli#secondMark` -> `func secondMark(seconds int) string`

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistBlockLine`
- `internal/loomcli#checklistRound`
- `internal/loomcli#checklistReviewBlock`
- `internal/loomcli#checklistVisitLine`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/statuspane#FormatSeconds`

**Intent:**
The checklist draws every closed span to the second and every time that includes the open visit in whole minutes (D1), and each review round shows its total (D2).

The open visit is the visit at history index -1, the current producer's running visit.
- `checklistVisitLine` draws a closed visit's time with `statuspane.FormatSeconds` and the open visit's with `statuspane.FormatMinutes`.
- `renderChecklist`'s row line draws the row total with `FormatSeconds` when none of the row's visits is the open one, else with `FormatMinutes`.
- `checklistReviewBlock`'s heading draws the segment total with `FormatSeconds` when no visit of the segment is the open one, else with `FormatMinutes`, and keeps its round count.
- Each round line reads `round <n> <time>`: the sum of the spent times of the visit lines placed in that round, a goto line adding nothing since it carries no time; the sum is drawn with `FormatMinutes` when the round holds the open visit, else with `FormatSeconds`.
  `checklistBlockLine` gains the line's `spent` and whether it is the open visit, and the round's sum is taken from its lines when the block is drawn, so a bounced Bouncer line that waits and opens the next round counts in that round.

No frame changes while only the open visit's next second passes: every seconds-precision value is closed and every value that grows is in whole minutes, so `statuspane.Pane` redraws, and wipes the scrollback, at most once a minute.

`TestRenderChecklist`'s wanted frames change their closed times to seconds (`5m00s`) and keep the open visit and every total including it in minutes; every round line shows its total.
Its new rows and subtests:
- a review block whose round 1 closed and whose round 2 holds the open visit draws `round 1 10m00s` and `round 2 <n>m`, with the segment heading in minutes;
- a finished review block draws its heading in seconds;
- visits whose ends fall on seconds draw `45s` and `12m05s` lines and a `1h05m12s` row total; their entries' `At` come from `secondMark`, a test helper in `checklist_test.go` beside `minuteMark` that renders the instant the given seconds after `checklistClock` as RFC 3339;
- a subtest renders one status at `now` and at `now` plus one second inside the same minute, and asserts the two frames are byte-identical.

The package doc's checklist sentence names the two time formats and the round totals.

**ImpactSummary:** the checklist frame's time text only, inside `internal/loomcli/checklist.go`; no verb, flag or record changes, and `TestRenderChecklist` is the one test asserting the frame.

**Commit:** 8: loom checklist: closed times to the second and round totals
