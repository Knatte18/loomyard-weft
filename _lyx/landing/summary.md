# Checklist boxes a row its entry gate skipped `[-]`, a halted row `[=]`

The loom checklist, the overview window's frame, showed a row the run passed on a false entry gate, such as a switched-off Drift row, as `[ ]`.
That reads as not yet reached, though shed had settled the row and moved on.

## What changes

- A row with an `OutcomeSkipped` history entry and no visit is boxed `[-]`, with no time and no visit count.
  This holds for every row an entry gate skipped.
- A review segment whose rows were skipped and never visited draws its single heading line as `[-] <number>. <segment>` instead of `[ ]`.
- The halted box, for a paused, blocked or awaiting current producer, moves from `[-]` to `[=]`.

## Design decisions

- Precedence is the current producer's state box, then `[x]` for any visit, then `[-]` for a skip, then `[ ]`.
  A row that was skipped and later visited, through a goto for example, shows `[x]`, since it has run and has work to show.
- The halted box changes glyph because the board entry fixes `[-]` for the skipped row, and one glyph for both states would read the same in one frame.
  `[=]` echoes a pause sign; the state line still names the halt.
- A visited review segment's heading takes only a current producer's state box, so a skipped row inside an otherwise visited segment never boxes the heading `[-]`.
- The skipped rows are read straight from the status history in `checklistSkippedRows`; `checklistVisitsOf` still treats a skip as no visit.

## Tests

`TestRenderChecklist` rows cover a skipped row boxed `[-]`, a row both skipped and visited boxed `[x]`, and a review segment that was only skipped boxed `[-]`.
The rows that pin a halted current producer now expect `[=]`.
`internal/loomcli/doc.go` names the skipped box in its checklist paragraph.
