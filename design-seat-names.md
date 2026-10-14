# Seat names: none for a lone seat, operator-chosen names for several, no chair

`seat-names` · Note · enhancement, shed, burler, reed

## Seen (2026-10-14)

Discussion and every Burler round run through the seat engine (`internal/seatengine`), even with one agent.
A seat's name is fixed: `seatengine.RoleChair` (`chair`) for the leading seat and `advisor-<n>` for the others (`Seat.Name` in `internal/seatengine/table.go`).
The name goes into the strand and role names the operator sees, such as `burler-review-chair` and `burler-fix-chair` (`internal/burlerengine/doc.go`), even in a round with one seat.
A recipe's MultiLLM row has a `name` key per seat, but the engine expects `chair` and `advisor-<n>`.

## Wanted (operator, 2026-10-14)

- A lone seat carries no seat name: the role reads `burler-review`, not `burler-review-chair`.
- With two or more seats, the operator names each seat in the recipe or config, and that name is what the windows, strands and overview show.
- `chair` is not a name to show the operator.

## To consider (operator, 2026-10-14)

A name that shows which round a session belongs to, since `burler-review-chair` does not say which round it is.
The operator notes this may matter less now that the overview window shows the rounds.

## To check

- Where `chair` and `advisor-<n>` are load-bearing: stencils (`seat-directive-chair`, the `advisor_names` marker), strand names, reed window names, and any lookup by role.
- Whether the leading seat is marked by its place in the table or by a flag, once the name is free.
