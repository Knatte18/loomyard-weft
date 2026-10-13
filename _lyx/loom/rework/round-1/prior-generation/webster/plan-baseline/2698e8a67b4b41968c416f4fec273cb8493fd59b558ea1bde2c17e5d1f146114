# Card 2 — checklist-frame

**Edit:**
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistVisits`
- `internal/loomcli#checklistVisitsOf`
- `internal/loomcli#TestRenderChecklist`

**Delete:**
- `internal/loomcli#fitChecklist`
- `internal/loomcli#checklistMoreRowsFormat`
- `internal/loomcli#TestRenderChecklist_Fit`

**Create:**
- `plan:internal/loomcli#checklistVisit` -> `type checklistVisit struct`
- `plan:internal/loomcli#checklistDetailLines` -> `func checklistDetailLines(name string, v checklistVisits) []string`
- `plan:internal/loomcli#checklistWaitLine` -> `func checklistWaitLine(poll statuspane.StatusPoll, waitNote string) string`

**Intent:** Redraw the loom checklist as D7's frame: detail blocks on top, then the row list, then the state line and the wait line last, so a terminal shorter than the frame keeps the list and the wait in view and the operator scrolls up for the detail.

`checklistVisit` is one visit of a row read from the status history, with no new persisted field: its start (the previous entry's end, as the visit times are today) and whether that start is known, its time spent, its outcome, its output path, its gate attempts (the entry's `GateAttempts`, nil when ungated), its fold count (`1+Repeats`) and whether it is the current producer's open visit.
`checklistVisits` gains a `log []checklistVisit` field, and `checklistVisitsOf` appends one visit per non-goto history entry and one open visit for the running current producer, with the same time arithmetic it already uses for `spent`.

`checklistDetailLines` renders one row's block: a header line `<row>` then one line per visit, indented two spaces, `<start> <spent> <outcome>`, then ` x<n>` for a folded entry, ` <output>` when the output path is non-empty and ` gate attempts <n>` when gated.
The start is the visit's start time formatted `15:04` in the zone the status recorded (UTC), or `--:--` when unknown, and the spent time is `statuspane.FormatMinutes`; an open visit's outcome reads `running`.

`checklistWaitLine` is the last line and shows the full reason `lyx loom status` gives, never a rewording of the `reviewWaitingFor` note alone:
a running run with a waiting note reads `waiting <note>`;
an awaiting run, the hand-off to the parent, reads `waiting on the parent: escalation from <current producer>`, followed by `: <parent notice>` when the status carries one and ` | <activity wait>` when it carries a reason;
a blocked or failed run reads `wait <activity wait>`, the field `lyx loom status` prints for a halt;
every other case, an unreadable status included, reads `waiting on nothing`.

`renderChecklist` composes, top to bottom: one `checklistDetailLines` block per visited row, recipe rows first in recipe order and history-only rows after; the row list exactly as today (box, time spent, `x<n>` visits, which for a review row are its rounds, `(not in recipe)`); the state line, now `shedverbs.RenderStatusLine` over the status with `Activity.Wait` cleared, since the wait line carries it, or the unavailable line; then the wait line.
With the recipe unreadable the row list is the single line `recipe unreadable`, still above the state and wait lines.
It returns the frame's `Lines` with no `Fit`, so `fitChecklist`, `checklistMoreRowsFormat` and `TestRenderChecklist_Fit` are deleted; until card 3 drops the pane's own row cut, a frame taller than the pane shows its first rows.
`checklist.go`'s file comment describes the new frame; card 3 rewrites `checklistCmd`'s `Long` text with it, and card 9 rewrites `internal/loomcli/doc.go`'s paragraph.

`TestRenderChecklist`'s want slices are rewritten to the new order for every existing row, and it gains rows:
"an awaiting run's wait line names the escalation to the parent" (state awaiting on `Plan-Bouncer` with a parent notice and an activity wait, wanting `waiting on the parent: escalation from Plan-Bouncer: <notice> | <wait>`);
"a blocked run's wait line carries the halt reason in full" (a reason longer than any terminal, wanting it whole);
"a visited row's detail lists each visit with its start, time, outcome, output and gate attempts" (a history entry carrying an output path and `GateAttempts`, a folded entry, and the current producer's open visit);
"a history-only row's detail follows the recipe rows' details".
The existing "a running run shows done, running and not reached rows in recipe order" row, a running run with no note, now ends with `waiting on nothing`, which pins that case.
The "a wait note follows the state line" row becomes "a running run's wait line carries the note in full", the note on the last line and absent from the state line.

**ImpactSummary:** One pane verb's frame layout changes; `renderChecklist` and `checklistVisitsOf` have no caller outside `checklist.go`, only `TestRenderChecklist` asserts their output, and the deleted fit helper's one caller is `renderChecklist` on this card.
