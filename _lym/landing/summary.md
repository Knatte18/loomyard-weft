# raddle: file tasks as drafts, order chains by the record, show deps

Raddle's proposal could hide its own order, and its filing released work the operator had not released.
In the `fabric-test-speed` run three things went wrong:

- The agent chained parts on one `internal/fabricengine` package against the Sketch's own order, and named neither the departure nor its cost.
- The table shown to the operator had no dependencies, so the order could not be read from it.
- Filing released the tasks of the `tasks`, `tapestry` and `project` shapes, so the dobby started them before the operator could make the project a beam.

This change fixes all three.

## What changes

- **Chain order.** The raddle stencil (`raddle-template-agent.md`) now says how to order parts that share a package and must be chained.
  The Sketch's own order comes first, and a part the record says waits for nothing goes first in its chain.
  Where the record states no order, the part with the largest stated saving goes first, or the one the others gain from earliest.
- **Order departures.** The proposal has an optional `order_departures` key, a list of `{parts, record_order, cost}`.
  The agent lists each place its order departs from an order the record states.
  `raddleparser.Proposal` carries it as `OrderDepartures`, and `raddle-proposal-spec.md` describes it.
  It is optional, so an existing proposal parses unchanged.
- **Approval prompt.** In interactive mode (`raddleModeRules` in `loomengine`) the agent shows the parts as a table with a column of each part's dependencies, by slug.
  It names each order departure with its cost.
  It says that every task lands as a draft, released with `lym board set-status`.
- **Drafts only.** Filing no longer releases anything.
  `RaddleFiling.Release`, `Board.ReleaseTasks`, `RaddleFileDeps.Release` and the `raddleFiled` mutation result are removed.
  `FileRaddle` leaves every task draft, project members included, and the parent notice says the tasks wait as drafts for the operator's release.
  The beam shape already filed drafts; it now differs from the others only in the `CreateBeam` step.
  `RaddleFile` loses its release step, so its remaining steps renumber.

## Design decisions

- Release is removed, not made optional.
  Approving a proposal is not a release, and no caller wants the old behavior.
- `order_departures` is a proposal key, not prose in the table.
  This keeps departures in the proposal file the operator approves, where the checker can later read them.
- The dependency column and the departure listing are instructions to the agent in the mode rules.
  Neither is enforced in code.

## To check by hand

- Run a raddle session in interactive mode and confirm that the pane table shows the dependency column, the departures and the draft notice.
  Only the prompt text and stencil changed, so no test covers the agent's behavior.
- Confirm that a filed project's tasks all read `draft` on the board, and that `lym board set-status` releases them.
- Run the `battencli` and `shedrecipe` tests, which pin the `RaddleFile` seam lists that lose `Release`.
