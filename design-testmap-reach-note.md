# Reach note for a diff: the tests reaching each changed function, and changed functions nothing reaches

`testmap-reach-note` · Note · feature, testmap, gate, burler

## What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 2) ranks this second and builds it first:
uses 1 and 3 read the maps through the same reader.
The code review rubric already asks for "a behavior the diff adds with no coverage" and that a redundancy finding name its covering test;
today the reviewer answers both by searching and reading.
A name search cannot find tests that reach a function through callers: 793 reached functions are reached by no test of their own package,
and half of all reached functions are reached from more than one test package.

## Idea

- A reader of the maps with an index (a lookup across packages reads up to 56 MB of JSON).
- A finder of the functions a diff changes, comparing function texts (the old one went with the selector in `72b26d1a`).
- A note in the code review: for each changed function the tests that reach it, own package first and most specific first, ranked and capped
  (153 functions are reached by 500 tests or more); and the changed functions no test reaches,
  saying when that may only mean a tier 2 or spawn-marked test runs it.
- The writer gets the same note before review: its verify gate prints, once the tiers pass, the changed functions no test reaches and the tests reaching each changed function, so the writer adds the missing tests itself and review mostly finds none left (operator, 2026-11-18).
- A read verb a writer can call for a function before it edits.
- No upkeep per landing: a stored map stays sound for days (98.5 % of a fresh map's pairs after three days), plus a fresh map of the diff's packages.

Study estimate: 2 darns.
