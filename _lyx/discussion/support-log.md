# Support log — checklist-skipped-row-box

## Interview

A darn run holds no interview; the writer read the board entry and the checklist code and settled the calls itself.

## Rejected alternatives

- Keep `[-]` for halted and give skipped rows another glyph: rejected, the board entry names `[-]` for the skipped row.
- Share `[-]` between halted and skipped: rejected, the two states would read the same in one frame.
- Box a row by its latest entry, so a skip after a visit reads `[-]`: rejected, a row that ran has work to show and `[x]` keeps that visible.

## Review rounds

_No rounds yet._

## Question ledger

- Which glyph does the halted box take once `[-]` means skipped? Self-picked: `[=]`.
- Does a skipped and later visited row read skipped or done? Self-picked: done, `[x]`.
- Does a review segment get the skipped box? Self-picked: yes, when no row of it was visited and one was skipped.
