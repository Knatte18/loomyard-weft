# lym gate redundant flags tests as redundant that check different things

`redundancy-judge-asserts` · Task · bug, testkit, gate · running · Code-Burler-Review

## Seen (operator, 2026-11-16)

`redundancy-candidates-survey` (`194650f0`, report `docs/research/speed/redundancy-candidates-survey-2026-11-16/`) listed 1,026 candidates in 118 packages,
and every candidate it sampled in the five largest packages asserts something its covering tests do not.
The pilot (`docs/research/speed/coverage-map-pilot-2026-11-12/report.md`) found the same.
The operator's ruling: a module that flags two tests as testing the same thing when they do not is defective, and is to be fixed, not worked around by reading.

## Cause

`internal/testcover` records which blocks each test executes, and `lym gate redundant` calls a test redundant when other tests execute all its blocks.
Executing a line is not checking it: two tests may run the same code and assert different results.

## Idea

Keep coverage as the cheap first filter, and confirm each candidate before it is listed:

- Mutation check: for each block the candidate covers, apply small mutations (flip a comparison, negate a condition, change a constant, drop a statement) and run the candidate and its covering tests against each mutant.
  The candidate is redundant only if every mutant it kills is also killed by a covering test.
- Run it per package on command, as the map build runs; its cost is bounded by the candidate's blocks, not the whole package.
- The research part settles whether lym writes its own mutator over `go/ast` or uses an existing Go mutation tool, and measures the cost on the survey's five largest packages.

The candidates the survey sampled are the yardstick: the fixed verb must drop each of them.
