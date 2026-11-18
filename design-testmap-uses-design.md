# Coverage map uses: reach note, failure triage and diff mutation check

`testmap-uses-design` · Task · sketch, testmap, gate, burler, darn · running · Raddle-Write

Decision record: [Sketch](decision-testmap-uses-design.md)

Design the three uses of the per-test coverage maps the map study ranked worth building (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, `884424fb`), as one design over one reader of the maps.
Build order: the reach note first, since the other two read through its reader; then failure triage; then the diff mutation check, which starts with its hand trial.

## Reach note for a diff: the tests reaching each changed function, and changed functions nothing reaches

### What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 2) ranks this second and builds it first:
uses 1 and 3 read the maps through the same reader.
The code review rubric already asks for "a behavior the diff adds with no coverage" and that a redundancy finding name its covering test;
today the reviewer answers both by searching and reading.
A name search cannot find tests that reach a function through callers: 793 reached functions are reached by no test of their own package,
and half of all reached functions are reached from more than one test package.

### Idea

- A reader of the maps with an index (a lookup across packages reads up to 56 MB of JSON).
- A finder of the functions a diff changes, comparing function texts (the old one went with the selector in `72b26d1a`).
- A note in the code review: for each changed function the tests that reach it, own package first and most specific first, ranked and capped
  (153 functions are reached by 500 tests or more); and the changed functions no test reaches,
  saying when that may only mean a tier 2 or spawn-marked test runs it.
- The writer gets the same note before review: its verify gate prints, once the tiers pass, the changed functions no test reaches and the tests reaching each changed function, so the writer adds the missing tests itself and review mostly finds none left (operator, 2026-11-18).
- A read verb a writer can call for a function before it edits.
- No upkeep per landing: a stored map stays sound for days (98.5 % of a fresh map's pairs after three days), plus a fresh map of the diff's packages.

Study estimate: 2 darns.

## Failure triage: does a failing test reach what the run changed

### What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 3):
all four flaky failures since 2026-11-01 that the maps hold were tests reaching no file their landing changed;
DeFlaker's rule would have called them flaky before any rerun.
Today a repeated failure that is not the run's own costs a round trip to the parent to test the base,
and on 2026-11-18 a fixer chasing such a failure ran the memory hog that crashed Neve.

### Idea

- For each failing test the maps hold, a line in the gate's failure report and in the fixer's brief:
  the run's changed functions it reaches, or "reaches none of this run's changes".
- For a parent branch gone red, the same lookup over the landings since the last green names the landings whose changes the test reaches.
- Reads through the reader of note `testmap-reach-note`.

Study estimate: 1 darn on top of the reach note.

## Mutation check of the code a diff changes, survivors to the code review

### What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 1) ranks this first:
the only use that sees what tests assert. A mutant no test kills is a changed behaviour no test checks.
Google runs this on changed lines in code review (Petrović et al.); developers first called 85 % of mutants unproductive, so a cap and a filter are part of the first build.
`internal/testcover` already plans mutants per block, builds them into one test binary and judges kills.
With the map each mutant runs the tests covering its block (2 at the median) instead of the whole package (23.8 on average).

### Idea

1. Trial first: replay the step by hand over about twenty landed diffs, count survivors and read a sample, to learn how often a landed diff holds an unchecked change.
2. If the trial pays: a gate report step after the tiers pass, mutants only in the diff's new or changed blocks, capped per diff,
   uncovered blocks reported as uncovered without mutants, survivors to the code review with file, line and operator.
   A survivor only another package's test could kill is discounted through the reach note (note `testmap-reach-note`).

Study estimate: a trial darn, then 3 to 4 darns.

