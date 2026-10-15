# Support log — tokencount-fixes

## Interview

A darn run holds no interview: the writer worked from the board entry and a reading of `tools/tokencount`, `internal/pauserecord`, `internal/hibernateengine` and the overview-seats commit.

## Rejected alternatives

- **Treat any gap with no transcript line in any session as sleep.**
  A machine awake with every run parked writes no transcript line either, so the heuristic would erase real idle time.
- **Have lyx persist the sleeps its `SleepWatch` sees.**
  The watches run only while a lyx process waits, so they miss sleeps between processes, and the change would reach far beyond `tools/tokencount`.
- **Map an explicit list of old role names.**
  Every seat table formed its lead role as `<prefix>-chair`, the generic MultiLLM recipe row included, so a closed list would miss a recipe-defined prefix.
- **Keep the share over the span and add an active column beside it.**
  The board entry asks for the share itself to rank by active time.

## Review rounds

_No rounds yet._

## Question ledger

- **Where do sleep times come from?** Self-pick: the systemd journal's `systemd-sleep` lines, which this host records for every hibernate and suspend.
- **What does the report do without a journal?** Self-pick: subtract nothing and say why in the header, so Windows and WSL runs still report.
- **What is the share's denominator?** Self-pick: the table's summed active time, which equals the run's working time inside the span.
- **Which roles does the rename map?** Self-pick: every role ending in `-chair`, dropped to its prefix.
