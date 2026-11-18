# Mutation check of the code a diff changes, survivors to the code review

`testmap-diff-mutation` · Note · feature, testmap, gate, burler

## What was seen

The map-uses study (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, use 1) ranks this first:
the only use that sees what tests assert. A mutant no test kills is a changed behaviour no test checks.
Google runs this on changed lines in code review (Petrović et al.); developers first called 85 % of mutants unproductive, so a cap and a filter are part of the first build.
`internal/testcover` already plans mutants per block, builds them into one test binary and judges kills.
With the map each mutant runs the tests covering its block (2 at the median) instead of the whole package (23.8 on average).

## Idea

1. Trial first: replay the step by hand over about twenty landed diffs, count survivors and read a sample, to learn how often a landed diff holds an unchecked change.
2. If the trial pays: a gate report step after the tiers pass, mutants only in the diff's new or changed blocks, capped per diff,
   uncovered blocks reported as uncovered without mutants, survivors to the code review with file, line and operator.
   A survivor only another package's test could kill is discounted through the reach note (note `testmap-reach-note`).

Study estimate: a trial darn, then 3 to 4 darns.
