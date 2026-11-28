# Measure where Webster and Weaver rows spend their gate time since the card gate took in dependent packages

`implementer-gate-time-measure` · Task · research, gate, webster, weaver · done

## Why

`docs/research/webster/webster-time-2026-11-08.md` named five levers worth 15 to 23 min of an 86 min Webster row, the largest the guard step in every card gate.
The Weaver bench of 2026-11-27 (`docs/research/weaver/bench-2026-11-27/report.md`) then found gate tier time per row 2.2 times its recorded figure,
the same day `a69580709` made the card gate also run the packages that depend on a card's packages.
Neither cause is measured, so the cuts would rest on guesses.

## What to measure

Read recorded data only: gate run records under the Bolt checkout's `.lym/gate/runs`, implementer state and row history, and the archived runs' records.
Start no run, agent, bench or gate call.

Take every Webster and Weaver implementer row that ended between 2026-11-20 and the task's start, split at `a69580709`'s landing.
For each, split the row's time into agent work, card gates, batch gates, plan-level verify, gate fixers and gate-slot waits.
Within the card gates, give the time of each step of the tier commands, the guard step and the dependent-package scope included.

## Report

A report under `docs/research/webster/` with those tables, before and after `a69580709`,
and for each of the 2026-11-08 levers whether it still holds and what it would save now.
Add any new lever the records show, and rank all of them by minutes saved per row.
