# tokencount: drop machine sleep, active-time shares, map -chair roles

## What changes

**Machine sleep is no time at all.**
`tools/tokencount` reads the machine's suspends and hibernates once per invocation from the systemd journal (`journalctl -t systemd-sleep -o json`).
Each sleep runs from systemd-sleep's `Performing sleep operation '<op>'...` entry to the next `System returned from sleep operation '<op>'.` entry of the same operation and boot.
`RunTally.subtractSleeps` cuts the sleeps out of every working interval, the run's and each role's, and the span, the idle row and each Producers row's elapsed and idle count awake time only.
Each run's section names the sleep left out of its span, "All runs" names the sum, and the report header says where the sleeps came from or why none were left out: `journalctl` not on `PATH`, failing, or holding no sleep, as on Windows.
A new `-no-sleep` flag reads no journal and counts sleep as time.

**Producers share by active time.**
The Producers table gains an `active` column, elapsed minus idle.
The share is a row's active time over the table's summed active time, and rows are ordered by active time, then by name, so a row parked through a long wait no longer tops the table.

**Pre-rename lead roles.**
`roleOf` drops a trailing `-chair` after the `-N` suffix, so `burler-review-chair`, `burler-fix-chair` and `discussion-chair` count under `burler-review`, `burler-fix` and `discussion`, and a report spanning the overview-seats rename sums each role once.
Advisor roles read `<prefix>-advisor` on both sides of the rename and need no mapping.

Over the last ten full loom runs on the operator's host, the `tender` run loses 22h12m of hibernation, and the All-runs Plan-Bouncer row falls from 50.5% to 1.6%, 29m51s elapsed.

The package doc in `tools/tokencount/main.go` describes the sleep source, the active-time share and the role mapping.
`tools/tokencount`'s test budget rises by one for `TestParseSleepJournal`.

## Design decisions

- The journal is the sleep source because lyx persists no sleep: `pauserecord.SleepWatch` lives in memory, only while a lyx process waits.
- A missing or failing journal is a header line, not an error, so a report still runs where there is no systemd.
- `-no-sleep` was added while coding: the integration test calls `run` and would otherwise read the host's real journal; it passes `-no-sleep` to stay hermetic.
- The `-chair` mapping is a suffix rule, not a list of names, because the generic MultiLLM recipe row formed `<prefix>-chair` for any recipe-defined prefix.
  A seat an operator names `chair` after the rename also folds into its table's lead role.

## To check by hand

- A machine powered off, rather than suspended or hibernated, during a run still counts as time, since the journal records no sleep operation for it.
- A journal whose retention no longer covers a run's window leaves nothing out for it; the run's section then prints no sleep line.
