# Support log: dobby-overview-window

## Interview

A darn run holds no interview: the scope is read from the board entry and the code.

## Rejected alternatives

- A window or strand of its own for dobby: the board entry asks for none.
- Rendering the view inside dobbyengine: the overview owns its line rendering in `battencli`, and the engine stays free of display concerns.
- Having dobby write its view to a file the pane reads: it would add a write per tick and still miss a dead dobby, which only a lock probe reads.
- Keeping the hub reads in `beamcli` and running the overview from `beamcli`: it would move the pane's command and its strand wiring for no gain.
- Showing only the first reason a task waits: the operator would fix one cause and find the next one only on the next look.
- `lyx beam status` for the acting worktree only: `lyx beam start` already reports per local dobby, and one verb showing them all matches it.

## Review rounds

_No rounds yet._

## Question ledger

- Which overview shows which dobby? Self-pick: the pane shows the dobby of the worktree it runs in, the prime or a beam.
- What does "last pass" mean? Self-pick: the exit time of the run's last pass from the state record, with a held driver lock as a pass in flight.
- Where does a refused start's reason come from? Self-pick: a new optional text in the start record, since none was kept.
- Which dobbies does `lyx beam status` print? Self-pick: the prime's and every local beam's.
