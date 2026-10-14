# Overview window: no stuck for a bounce, log and checklist told apart

`overview-window-tidy` · Note · enhancement, reed, shed

## Seen (2026-10-14, `tender`'s overview window)

- The window shows a log (one line per attempt, with time, outcome and file) and below it a checklist (one line per recipe row, `[x]`, `[>]`, `[ ]`, with total time and attempt count).
  A finished row so stands twice, and the operator read it as a duplicate.
- Plan-Bouncer showed `22h18m`: the pause and the overnight hibernate count as the row's time, though the row itself took 0 minutes.
- Bouncer and Burler rows that bounce to the next round read `stuck`.

## Wanted (operator, 2026-10-14)

- No `stuck` for a bounce: the state machine is not stuck, a bounce is part of the normal flow; `stuck` would mean something sits fast.
  Rename the outcome where it is shown (and consider the name in the shed recipe's `OnStuck` too), for instance `bounced`.
- Make it clear which part is the log and which the checklist, or drop finished rows from the checklist since the log holds them.

## To consider

- Paused time (`lyx pause`, hibernate) should not count as a row's time.
