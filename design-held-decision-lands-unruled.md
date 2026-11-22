# A decision finding held for the parent lets the run land unruled

`held-decision-lands-unruled` · Note · bug, burler, loom

## Seen

`per-task-settings-one-lookup` landed as `118bee8ad` with its Code-Burler finding F12, `member-line-overrides-explicit-off`, never ruled on.
F12 was a `decision`-class finding, routed to the run's parent and marked held: the fixer leaves it alone, and only a decision record entry settles it.
The orch had put it to the operator, whose answer was still pending.

Round 2's Bouncer verdict states it: "F12 is a `decision` finding deferred to the parent and does not gate."
The round's circling decision then accepted on budget (`decision: accept`, `cause: budget`, `automatic: true`), and the run went on to land.
The held finding ended in the change description's "Check by hand" list and in the decision record's open risks, with the exception it questioned still in the code.

## Wanted

A held decision finding gates until the parent rules on it.
No acceptance, budget or otherwise, lets a run past its review round while a held finding is unruled.
The run waits for the parent's decision record entry, then the next round carries the ruling out.

## Where

- `internal/burlerengine`: the held mark (`roundcheck.go`, `prompt.go`, `fixerreport.go`).
- The Bouncer's gating rule that leaves decision findings out.
- The circling decision in `internal/loomcli/circling.go` that accepts on budget.
