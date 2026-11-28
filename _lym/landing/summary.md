# Measure where implementer rows spend their test-run time

Adds `docs/research/webster/gate-time-2026-11-28.md` and its line in `docs/research/README.md`.
No code changes.

The report splits eleven Webster and Weaver rows, ended 2026-11-20 to 2026-11-28, from recorded data only:
five before `a69580709` made the card check run dependent packages, six after it.

Findings:

- Checks, fixers and slot waits cost 7.1 min per card after the change, against 4.8 min before it.
  The two sides hold different plans.
- The card check's listed test run went from 5.5 s to 44.8 s per run.
- The batch check went from 6.6 to 24.1 min per row;
  none of its tier1 test failures after the change was in a dependent package.
- Dependent packages do catch failures at the card check: 10 of 23 failed listed runs outside fixer sessions.
- Cold fixers take 21.9 min per row, 12.6 min of it running the whole card check before it is run again to judge.

The report states whether each of the five levers of `webster-time-2026-11-08.md` still holds, adds four,
and ranks them by estimated minutes saved per row.
The first three are: batch tier1 tests over the batch's own packages (14.6 min),
the card check over the card's own packages (9 to 14 min),
and card fixers re-running only the failed check (about 11 min).
They overlap, and every saving is an estimate from recorded run times, none of it tried.

## Design decisions

- The bench runs of 2026-11-27 count as rows; they are four of the six after the change.
- A row's time leaves out machine hibernation and the time it stood blocked.
- The build, vet and test split inside a tier1 run is estimated from per-package test times,
  since the run records time the command as a whole.

## Merge danger

**Door:** two-way
**Blast radius:** docs
