# A Sketch opened from the board shows only its brief, with no link to its decision record

`board-sketch-shows-record` · Task · bug, board · running · Drift-Await

## Seen (operator, 2026-11-07)

Opening a Sketch from the board, as `S1` `fabric-test-speed`, shows its design page, `design-<slug>.md` in the Bolt checkout, which holds only the entry's body: the brief the Motif started from, 31 lines.
The decision record the Motif approved, the substance of a Sketch, sits beside it as `decision-<slug>.md` (288 lines) and nothing on the board's README or the design page links to it.
The operator found the Sketch "very short" and could not reach the record from the board.

The design and decision file names are in `internal/boardengine/replay.go`.

## Wanted

- A discussed entry's design page links to its decision record, near the top, and the README row of a discussed entry links to it as well, so a Sketch's record is one tap away on a phone.
- The same holds for any discussed task, not only a Sketch, and for a task a Sketch is promoted to.
- A link to a record file that is missing is not rendered.
