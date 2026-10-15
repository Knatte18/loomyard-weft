# Run status pane: seconds, Burler round totals, gate time and gate rework

`run-pane-timing` · Task · enhancement, loom, burler

Depends on: [`shed-tackler`](design-shed-tackler.md)

## Seen

The per-run status pane (the checklist each run's window shows, rendered through `internal/statuspane` and `internal/loomcli/checklist.go`) gives each row's time in whole minutes (`statuspane.FormatMinutes`).
A Burler review round shows its parts (review seats, fix) each with its own time, but no total for the round.
Gate refusals, and the rework they cause, do not show at all.

## Wanted

Operator's requests (2026-10-15), each in scope:

1. Show seconds as well as minutes for each row's time.
2. Give each Burler round a total time for the round, beside the time of each part.
3. Show the gates in the pane, with the time each gate took, and how much time went to rework because a gate refused (the rerun of the producer the gate sent back).

If item 3 cannot be read from the records a run keeps today, the writer says what is missing in the PR description and ships items 1 and 2 with as much of item 3 as the records allow.
