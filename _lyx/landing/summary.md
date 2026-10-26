# Checklist boxes a row skipped by its entry gate as [-]

The loom checklist overview left a row open, `[ ]`, after the run passed it on a false entry gate, such as the Drift rows with Drift switched off.
An open box reads as not yet reached, though the row was settled.
A row the run skipped and never visited is now boxed `[-]`, for every row an entry gate skipped.
The skip is read from the `OutcomeSkipped` entries of the status history through `shedengine.Status`; no new parsing and no change to shed's skip mechanics.

The halted box, the box of a paused, blocked or awaiting current producer, was `[-]` and moves to `[=]`.
Two states sharing one glyph would read the same in the frame.

## Design decisions

- Precedence is current producer's state box, then visited `[x]`, then skipped `[-]`, then not reached `[ ]`.
  A row that ran at least once has work to show, so a skipped-and-visited row stays `[x]`; a current producer with a skip entry keeps its running, failed or halted box.
- A review segment never visited but holding a skipped row, such as one whose review budget is 0, draws as the single line `[-] <number>. <segment>`.
- A visited review segment takes only a current producer's state box for its heading, so a skipped row inside it never changes the heading.

## Changes

- `internal/loomcli/checklist.go`: `checklistBoxSkipped`, `checklistBoxHalted` moved to `[=]`, `checklistSkippedRows`, and the skipped state in `checklistBox` and `checklistReviewBlock`.
- `internal/loomcli/checklist_test.go`: rows in the existing `TestRenderChecklist` table for the skipped box, the skipped-and-visited row, the skipped review segment, a visited segment with a skipped row, and a current producer with a skip entry in each state; the halted expectations move to `[=]`.
- `internal/loomcli/doc.go`: the checklist paragraph names the skipped box.

## Check by hand

An operator used to `[-]` for a halted row may misread `[=]` until it is familiar; the state line still names the halt.
No live run was observed with the Drift rows switched off.
