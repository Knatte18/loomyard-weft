# Decision record — tokencount-fixes

## Goal

`tools/tokencount` reports time a run spent in machine sleep as no time at all, ranks the Producers share by active time, and sums each Burler and seat-table lead role once across the overview-seats rename.

## Scope

- `tools/tokencount` only: its sleep source, its span, idle, role-time and Producers figures, its role mapping, its package doc and its tests.
- In scope: every time figure the report prints, in the role tables and the Producers tables, per run and over all runs.
- Out of scope: any lyx module, a durable sleep record written by lyx, the orch section's charging, the token and cost figures.

## Decisions

- **Sleeps come from the systemd journal.**
  tokencount reads `journalctl -t systemd-sleep -o json` once per invocation and pairs each `Performing sleep operation '<op>'...` line with the next `System returned from sleep operation '<op>'.` line of the same boot into one sleep interval.
  Rationale: lyx records no sleep durably (`pauserecord.SleepWatch` lives in memory), and the journal records every suspend and hibernate with its start and end on the operator's Linux host.
- **A missing journal is a note, not an error.**
  With `journalctl` absent, failing, or holding no such lines, the report subtracts nothing and its header says why.
  Bound: the report then counts sleep as before, as on Windows, and says so instead of hiding it.
- **Sleep is subtracted from every time figure.**
  Each working interval loses the parts a sleep covers; the span, each Producers row's elapsed and every idle figure count awake time only.
  The report states the sleep subtracted per run and over all runs.
- **The Producers share is by active time.**
  A row's active time is its elapsed minus its idle, printed as its own column; the share is the row's active time over the table's summed active time, and rows are ordered by active time, longest first, then by name.
- **A pre-rename lead role maps onto its new name.**
  A role ending in `-chair`, the lead role every seat table carried before overview-seats, counts under the role with that suffix dropped, so `burler-review-chair` counts as `burler-review` and `discussion-chair` as `discussion`.
  Advisor roles need no mapping: `<prefix>-advisor-<n>` reads `<prefix>-advisor` on both sides.
  Bound: a seat an operator names `chair` after the rename also folds into its table's lead role.

## Constraints

- `PATTERN-test-speed` and `PATTERN-test-economy`: the parsing and arithmetic are unit-tested without a real journal; tests extend existing tables where one fits.
- `PATTERN-comment-line-breaks`: Go comments one sentence per line.
- `PATTERN-spawn-observability` does not reach `tools/`, so the `journalctl` spawn needs no logger.
- The package doc in `main.go` carries the tool's design and changes with it.

## Auto-mode assumptions

- A darn run holds no interview; the operator's board entry is the whole statement of intent.
- The operator runs tokencount on the Linux host the runs ran on, so its journal holds the sleeps that matter.

## Open risks

- A machine powered off, not suspended, during a run is still counted as time: the journal records no sleep operation for it.
- A journal whose retention dropped the run's window subtracts nothing for that window, silently beyond the per-run sleep figure reading zero.

## Acceptance criteria

- A journal with a `Performing`/`returned` pair yields one sleep interval; an unpaired `Performing` line, a pair split across boots and unrelated lines yield none.
- A run whose span holds a sleep reports span, idle and Producers elapsed and idle net of the sleep, and a working interval straddling a sleep loses the slept part.
- The Producers share divides each row's active time by the summed active time, and a long idle row no longer tops the table.
- `burler-review-chair`, `burler-fix-chair` and `discussion-chair` titles count under `burler-review`, `burler-fix` and `discussion`.
- With no journal, the report header names why no sleep was subtracted.
