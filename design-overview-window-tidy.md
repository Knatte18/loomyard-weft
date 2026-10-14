# Overview window: one combined log and checklist, no stuck for a bounce

`overview-window-tidy` · Note · enhancement, reed, shed, loom

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

## Seen as well (operator, 2026-10-14)

- The log groups lines by recipe row, so Discussion-Bouncer's seven lines stand before all Burler-Review and Burler-Fix lines, though the three rows took turns, round by round (bouncer, review, fix, bouncer, ...).
  The order the run took is lost, and the log is hard to follow.
- The arbiter is not in the log at all: round 3's circling escalation and its `continue` decision do not show.

A view that fits both: within a review segment, lines grouped by round (round 1: bouncer, review, fix; round 2: ...), with the arbiter's spawn and decision shown in the round it decided.
This refines the combined view below: a segment's rows fill in round by round, not row by row.

## To consider

- Paused time (`lyx pause`, hibernate) should not count as a row's time.
