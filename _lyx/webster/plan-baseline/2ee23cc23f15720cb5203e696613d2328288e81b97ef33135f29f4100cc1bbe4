# Card 10 — checklist-gate-lines

**Create:**
- `plan:internal/loomcli#checklistGateLines` -> `func checklistGateLines(spans []shedengine.GateSpan, paused []pauserecord.Interval) []string`

**Edit:**
- `internal/loomcli#checklistVisit`
- `internal/loomcli#checklistVisitsOf`
- `internal/loomcli#checklistVisitLine`
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistBlockLine`
- `internal/loomcli#checklistReviewBlock`
- `internal/loomcli#appendChecklistBlockLines`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/shedengine#GateSpan`
- `internal/shedengine#GateSpanKindRework`

**Intent:**
Under a visit line whose history entry carries gate spans, the checklist draws one line per gate entry with a span (D6): `gate <name> <gate time>`, then ` refused <n> rework <rework time>` when that entry re-prompted.

`checklistGateLines` groups the spans by their `Gate` name, in the order each name first appears, and draws one line per name.
The gate time is the sum of that name's gate and held spans, and the rework time the sum of its rework spans; `n` is its count of rework spans, one per re-prompt sent; the line has the refused part only when `n` is above zero.
Each span counts its end less its start, less the pause record's coverage within it (`pauserecord.Excluded`, the visits' rule), clamped at zero, so a span ending before it starts counts zero.
A span whose start or end does not parse counts zero.
Every span is closed, so both times are drawn with `statuspane.FormatSeconds`.
The gate name is drawn as recorded, whether or not a recipe names it.
An empty or nil span list gives no line.

`checklistVisitsOf` computes each visit's gate lines from its history entry's `GateSpans` and `in.paused` into a new `checklistVisit` field `gateLines`, and records whether the entry carried a span list; the open visit has none.
`checklistVisitLine` keeps today's ` gate attempts <n>` suffix only for a visit with `gate_attempts` and no span list, the legacy shape; a visit with a span list draws its gate lines instead.
The gate lines stand right under their visit line, two spaces deeper, before the arbiter lines placed after that entry: `renderChecklist` draws them under a row's visit lines and under the recipe-unreadable list's lines, and `checklistBlockLine` carries them so `appendChecklistBlockLines` draws them under a review block's lines.

`TestRenderChecklist` gains rows:
- a visit with spans of two entries draws one line per entry in first-appearance order, the refused one as `gate plan 12s refused 2 rework 3m05s`, the other with no refused part;
- a held span's time adds into its entry's gate time;
- a pause interval inside a held span, the pending gate's, is left out of that span's time;
- a span whose end is before its start draws `0s`, and a span whose start does not parse adds nothing beside a parsable span of the same entry;
- an empty span list beside `gate_attempts` draws no gate line and no suffix; the existing row "a visited row's lines show each visit with its start, time, outcome, output and gate attempts", whose entry carries only `gate_attempts`, stays the pin of today's ` gate attempts <n>` suffix;
- a review block's visit with spans draws its gate lines under the visit line, inside its round;
- with the recipe unreadable, a visit with spans draws its gate lines under its line in the history-order list.

The package doc's checklist sentence names the gate lines.

**ImpactSummary:** frame text in `internal/loomcli/checklist.go` only; the legacy suffix stays for entries written before card 3, so in-flight runs draw as today.

**Commit:** 10: loom checklist: a gate line per gate entry under its visit
