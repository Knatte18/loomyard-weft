# Failure triage: does a failing test reach what the run changed

`testmap-failure-triage` · Note · feature, testmap, gate, darn

## What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 3):
all four flaky failures since 2026-11-01 that the maps hold were tests reaching no file their landing changed;
DeFlaker's rule would have called them flaky before any rerun.
Today a repeated failure that is not the run's own costs a round trip to the parent to test the base,
and on 2026-11-18 a fixer chasing such a failure ran the memory hog that crashed Neve.

## Idea

- For each failing test the maps hold, a line in the gate's failure report and in the fixer's brief:
  the run's changed functions it reaches, or "reaches none of this run's changes".
- For a parent branch gone red, the same lookup over the landings since the last green names the landings whose changes the test reaches.
- Reads through the reader of note `testmap-reach-note`.

Study estimate: 1 darn on top of the reach note.
