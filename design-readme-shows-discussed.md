# Board README marks a discussed entry and links its Sketch

`readme-shows-discussed` · Task · enhancement, board · done

The board README does not show which entries carry an approved decision record (their Sketch), so the operator cannot see that a `motif` run has finished an entry's Discussion (operator, 2026-11-03).

## Expected

- The README marks every entry flagged `discussed` in its row, in every table that lists it, and links the mark to the stored record `decision-<slug>.md` beside `board.json`, as the slug cell links the body `design-<slug>.md` today.
- An entry not flagged discussed renders as today.
- The mark is short, for example a `Sketch` link in the title cell's bullet lines or beside the recipe; the writer picks the place that keeps the row narrow and states the choice.
- A test pins both renderings.
