# tokencount: subtract hibernation, share by active time, map renamed Burler roles

`tokencount-fixes` · Task · bug, burler · running · PR-Gate

## Scope

Two fixes to `tools/tokencount`, both in how its report sums sessions.

### 1. Hibernation and idle time in producer shares

In `tools/tokencount/tokencount.sh -last 10` (2026-10-15), the All-runs Producers table put Plan-Bouncer at 50.5% share.
All of it came from one run, `tender`: one Plan-Bouncer session spanned 22h20m, of which 22h18m was idle, most likely the machine hibernating overnight while the run waited at a stop.
In every other run Plan-Bouncer took 1–8 minutes.

- Subtract hibernation from every elapsed and idle figure: a wall-clock gap where the machine was asleep is no time at all, not idle time.
- Rank the `share` column by active time (elapsed minus idle), not by elapsed, so a parked session cannot dominate the table.

Operator view, not a decision: hibernation times must be subtracted.

### 2. Burler role names across the overview-seats rename

`overview-seats` (PR #624) renamed the Burler seat roles; its decision record names `tools/tokencount` role keys as an open risk.
`tools/tokencount` takes a session role from the strand title, so runs before and after the rename may land in separate rows (e.g. `burler-review-chair` against the new name).
tokencount maps the old role names onto the new ones, so a report over runs on both sides of the rename sums each role once.
