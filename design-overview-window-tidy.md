# Overview window: one combined log and checklist, no stuck for a bounce

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

## Idea (operator, 2026-10-14)

One combined view instead of two parts: the checklist in recipe order, and under each row its log lines (time, outcome, file) fill in as the row runs.
Rows still to run stay as bare `[ ]` lines, so the view shows both what happened and what is left.

## To consider

- Paused time (`lyx pause`, hibernate) should not count as a row's time.
