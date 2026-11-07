# Raddle files drafts, orders by the record, and shows dependencies

Raddle had three faults, now fixed:
it released the tasks it filed,
it chained parts that share a package against the order the Sketch's decision record states, without saying so,
and the table it showed for approval had no dependencies column.

## Filed tasks stay draft

Raddle-File no longer releases anything.
`boardengine.RaddleFiling` loses its `Release` flag, and `FileRaddle` files every part as a draft, whatever the shape.
The beam shape's release step and the `Release` seam of `loomshed.RaddleFileDeps` are gone, and so is `Board.ReleaseTasks`, whose only caller was that step.
The release stays with the operator, through `lym board set-status`, so a project can be made a beam before any of its tasks starts.
The filed notice to the parent now says that every filed task waits as a draft until the operator releases it.
The board and loomshed package docs and the weaving glossary's `draft` rows are updated to match.

## Order and departures

The raddle stencil's Step 3 now orders a chain of parts that share a package by the Sketch's own order first.
A part the record says waits for nothing goes first in its chain.
Where the record states no order, the part with the largest stated saving, or the one the others gain from earliest, goes first.

The proposal has a new optional key, `order_departures`, a list of `{parts, record_order, cost}`.
It has one entry for each place the proposal's order departs from an order the record states.
`internal/raddleparser` declares it as `OrderDeparture`, and `raddle-proposal-spec.md` pins it.

## The approval table

In interactive mode the mode rules have the agent show the parts as a table with a column of each part's dependencies, by part slug.
The agent names each listed order departure with its cost,
and says that every task lands on the board as a draft for the operator to release.

## Design decisions

- `ReleaseTasks` is deleted rather than kept unused: nothing else releases a set of tasks, and `lym board set-status` covers the operator's release.
- The departure list is written by the agent and not checked by Go: no parser can tell from the record's prose which order it states, so it has no check.
- The approval table is drawn by the agent from the mode rules and not rendered by Go, because the agent already writes the rest of the proposal as plain text in the pane.

## Tests

- `TestFileRaddle` asserts that every filed task is draft in each shape, and the redundant beam row and `TestReleaseTasks` are removed.
- `TestRaddleFile` drops the release step from the beam shape's step order, and asserts that the filed notice says the tasks wait as drafts.
- `TestParse_OptionalKeys` replaces `TestParse_PatternClusters` and pins the JSON form of both optional keys, `pattern_clusters` and `order_departures`.
