# Run status pane: numbered rows, seconds, round totals, gate time; segment-named sessions

`run-pane-timing` · Task · enhancement, loom, burler, shed, reed · running · Merge-Burler-Fix

## Seen

The per-run status pane (the checklist each run's window shows, rendered through `internal/statuspane` and `internal/loomcli/checklist.go`) gives each row's time in whole minutes (`statuspane.FormatMinutes`).
A Burler review round shows its parts (review seats, fix) each with its own time, but no total for the round.
Gate refusals, and the rework they cause, do not show at all.

## Wanted

Operator's requests (2026-10-15), each in scope:

1. Show seconds as well as minutes for each row's time.
2. Give each Burler round a total time for the round, beside the time of each part.
3. Show the gates in the pane, with the time each gate took, and how much time went to rework because a gate refused (the rerun of the producer the gate sent back).
4. Number the pane's entries, for every shed-driven run (loom, darn, batten and any later recipe), not only loom.
   A shed recipe has a fixed list of rows, so each row gets its number.
   A perch (a Bouncer row with its Burler rounds) is one row of the recipe and so takes one number; its bounces are rounds within that entry, not new numbers.
5. Give every spawned session a name that says which segment it serves.
   Today the Bouncer and Burler sessions of Discussion, Plan and Webster carry the same names (for example strand `ly:twill-fork0:bouncer-seed`), so they cannot be told apart.
   The segment goes into the name both in the pane and in the reed strand and window names the operator switches between with Alt+Up/Down.

Items 4 and 5 were added by the operator on 2026-10-15.

If item 3 cannot be read from the records a run keeps today, the writer says what is missing in the PR description and ships items 1 and 2 with as much of item 3 as the records allow.
