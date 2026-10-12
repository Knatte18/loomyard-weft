# Card 10 — a seat table declares one window group

**Create:**
- `plan:internal/seatengine#tableWindow` -> `func tableWindow(table Table) string`

**Edit:**
- `internal/seatengine#seatSpec`
- `internal/seatengine/doc.go`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`

**Uses:**
- `internal/seatengine#Table`
- `internal/reedengine/render#AnchorOwnWindow`

**Intent:**
D11: the producer, not the strand, is the unit a window is shared by, so a table's seats declare one group and reed's cap (card 5) shows a two-seat table together and a large table apart.

- `tableWindow(table)` returns the table instance's group name: `table.RolePrefix`, followed by `-` and `table.Round` when the round is set, so two rounds of one table never name the same group.
  It returns "" for a table of one seat, which then gets a window to itself named by its seat's role, as burler's one-seat reviewer and fixer tables do.
- `seatSpec` sets `Display{Anchor: render.AnchorOwnWindow, Window: tableWindow(table), WindowSize: len(table.Seats)}` on every seat, replacing the sharing size it set before; the chair and the advisors of one table carry the same group and size, so their arrival order never changes the arrangement.
  Its doc comment's sentence on the sharing size and the empty anchor shuttle turns into below-parent is replaced whole by one saying every seat of a table declares the `own-window` anchor with the table's window group and seat count.
- The table ends its seats' windows through the stops it already runs: the chair's done run is cleaned up by `Wait`, and `join` stops the chair when needed and every advisor, live or dead, before the step returns, which card 8 extends to dead own-window strands.
- `internal/seatengine/doc.go`'s sentence on seat strands says every seat declares the table's window group, named by its role prefix and round, and its seat count, so reed shares one window for a small table and gives each seat its own for a large one; a one-seat table gets its own window.

`TestSeatPrompts_ValuesPromptsAndSpecs` replaces the sharing-size assertion: for the three-seat table with round `r7` and prefix `multi`, both the chair's and an advisor's `Display` equal `{Anchor: own-window, Window: "multi-r7", WindowSize: 3}`.
It also asserts that the same table with its round cleared names the group `multi`, and that a one-seat table (the chair alone) yields an empty `Window` with `WindowSize` 1.

**ImpactSummary:** One seatengine spec field changes for every table; burler's one-seat tables get windows of their own, and multi-seat tables share up to reed's cap.

**Commit:** 10: Seat tables declare one window group sized by their seat count
