# tokencount counts hibernation and idle time in producer shares

`tokencount-active-time` · Note · bug

## Seen

In `tools/tokencount/tokencount.sh -last 10` (2026-10-15), the All-runs Producers table put Plan-Bouncer at 50.5% share.
All of it came from one run, `tender`: one Plan-Bouncer session spanned 22h20m, of which 22h18m was idle, most likely the machine hibernating overnight while the run waited at a stop.
In every other run Plan-Bouncer took 1–8 minutes.

## Idea

- Subtract hibernation from every elapsed and idle figure: a wall-clock gap where the machine was asleep is no time at all, not idle time.
- Rank the `share` column by active time (elapsed minus idle), not by elapsed, so a parked session cannot dominate the table.

Operator view, not a decision: hibernation times must be subtracted.
