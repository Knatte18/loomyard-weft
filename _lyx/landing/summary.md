# tokencount: skip machine sleep, share by active time, map -chair roles

`tools/tokencount` counted machine sleep as session time, ranked the Producers share by elapsed time, and split a Burler role across the overview-seats rename.
One parked `tender` Plan-Bouncer session spanned 22h20m, of which 22h18m was the machine hibernating, and it took 50.5% of the All-runs Producers table.
Roles recorded before the rename (`burler-review-chair`) and after it (`burler-review`) landed in separate rows.

## What changes

- **Sleep is subtracted from every time figure.**
  Each working interval loses the parts a sleep covers, so the span, each Producers row's elapsed and every idle figure count awake time only.
  The report prints the sleep subtracted per run, and over all runs only when sleep was subtracted.
- **The Producers share is by active time.**
  A row's active time is its elapsed minus its idle, printed as its own column.
  The share is the row's active time over the table's summed active time, and rows are ordered by active time, longest first, then by name.
- **A pre-rename lead role maps onto its new name.**
  A role ending in `-chair` counts under the role with that suffix dropped, so `burler-review-chair` counts as `burler-review` and `discussion-chair` as `discussion`.
  Advisor roles need no mapping.
- The package doc in `main.go` carries the new design.

## Design decisions

- **Sleeps come from the systemd journal.**
  `sleep.go` reads `journalctl -t systemd-sleep -o json` once per invocation.
  It pairs each `Performing sleep operation '<op>'...` line with the next `System returned from sleep operation '<op>'.` line of the same boot into one interval.
  lyx records no sleep durably, and the journal holds every suspend and hibernate with start and end.
- **A missing journal is a note, not an error.**
  With `journalctl` absent, failing or holding no such lines, nothing is subtracted and the report header says why.
  The report then counts sleep as before, as it does on Windows.
- **Journal parsing is separate from outcome mapping**, so both are unit-tested without a real journal.
- A seat an operator names `chair` after the rename also folds into its table's lead role.

## Limits

- A machine powered off, not suspended, during a run is still counted as time, since the journal records no sleep operation for it.
- A journal whose retention dropped the run's window subtracts nothing for that window; the per-run sleep figure reads zero.

## To check by hand

- Run `tools/tokencount/tokencount.sh -last 10` on the Linux host and confirm the `tender` Plan-Bouncer row no longer tops the All-runs Producers table.
- Run it with `journalctl` off `PATH` and confirm the header names why no sleep was subtracted.
- Run it over runs on both sides of the rename and confirm each Burler and discussion role has one row.
