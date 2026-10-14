# Card 4 — checklist-combined-frame

**Create:**
- `plan:internal/loomcli#checklistInput` -> `type checklistInput struct`
- `plan:internal/loomcli#checklistVisitLine` -> `func checklistVisitLine(visit checklistVisit, row string) string`

**Edit:**
- `internal/loomcli#renderChecklist` -> `func renderChecklist(in checklistInput, now time.Time) statuspane.Frame`
- `internal/loomcli#loomCLI.checklistPoll`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/loomcli#checklistVisit`
- `internal/loomcli#checklistVisitsOf`
- `internal/loomcli#stuckEntry`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/doc.go`
- `//docs/overview.md`

**Uses:**
- `plan:internal/loomrecipe#Rows`
- `plan:internal/loomrecipe#RecipeRow`

**Intent:** D1, and the outcome words of D3 and D4, in `internal/loomcli/checklist.go`.

The renderer takes one input value so later cards add inputs without re-signing it: `checklistInput` holds `rows []loomrecipe.RecipeRow`, `rowsErr error`, `poll statuspane.StatusPoll`, `waitNote string` and `startedAt string`, and `renderChecklist(in, now)` replaces the six-argument form.
`checklistPoll` reads the rows with `loomrecipe.Rows(seed.Recipe)` in place of `RowNames`, and builds the input.
The renderer stays pure over its input (PATTERN-test-speed); the checklist reads no wait mark (PATTERN-wait-mark-display-only).

One combined frame replaces the detail blocks above the row list:
- One box line per recipe row in recipe order, `<box> <row>` followed, when visited, by its summed time and ` x<n>` for more than one visit, exactly as the row list reads today.
- Under a visited row's box line, one line per visit, indented two spaces: `<start> <time> <outcome>`, then ` x<n>` for a folded entry, ` <output>` when it has one and ` gate attempts <n>` when gated, as `checklistDetailLines` drew them; `checklistVisitLine` renders one such line, prefixing the row name before the outcome when told a row.
- A row never reached stays one bare `[ ]` line.
- History-only rows follow the recipe rows, box line marked `(not in recipe)`, with their visit lines under them.
- With the recipe unreadable (`rowsErr` set), the frame shows `recipe unreadable`, then every visit line in history order, each naming its row through `checklistVisitLine`.
- The state line and the wait line stay last.
`renderChecklist` stops calling `checklistDetailLines`, since no detail block is drawn any more; card 7 deletes it once no later checklist card edits code that names it.

Outcome words, decided in `checklistVisitsOf`, which fills a new `row` field on each `checklistVisit` so the unreadable-recipe list can name it:
- A Stuck entry whose `RoutedTo` is set (card 1) reads `bounced`; a Stuck entry without it (a halt, or one an older binary wrote) reads `stuck`.
- An `awaiting` entry reads `escalated`.
- Every other outcome reads as its stored value; the open visit reads `running`.
Only the checklist shows these words; the status file, `lyx loom status` and the batten overview are unchanged.

The `checklist` verb's `Long` help, the checklist paragraph of `internal/loomcli/doc.go` and the `lyx loom checklist` mention in `docs/overview.md`'s loom bootstrap paragraph describe the combined frame: recipe rows in order with their visit lines under them, then the state and wait lines.

`stuckEntry` gains a routed-to argument ("" for a halt).
`TestRenderChecklist`'s rows are rewritten to the combined frame, and gain rows for:
- A frame part-way through a run: finished rows with their visit lines under them, the running row with its open visit, and every later row a bare `[ ]` line, no row drawn twice.
- A bounced Stuck entry (`routed_to` set) reading `bounced` beside a halted one reading `stuck`.
- An awaiting entry reading `escalated`.
- A history-only row after the recipe rows, marked `(not in recipe)`, with its visit line.
- An unreadable recipe: `recipe unreadable`, then the visit lines in history order, each naming its row, then the state and wait lines.

**ImpactSummary:** Confined to the checklist verb's file, its one test and two doc paragraphs; renderChecklist's sole caller is checklistPoll in the same file.
