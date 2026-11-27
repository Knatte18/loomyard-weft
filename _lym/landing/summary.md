# Report: what the coverage maps and their users give, and which to remove

Adds `docs/research/testmap-worth-evaluation.md`, its scripts and citation data under `docs/research/testmap-worth-evaluation/`,
and an entry in `docs/research/README.md`.
No code changes and nothing is removed: removal is a task of its own on the operator's word.

## Recommendation in the report

Remove every user of the stored maps (the `reach`, `mutation` and `redundancy` report steps with the reach turn and review markers,
`lym gate reach`, `mutate`, `redundant` and `test --map`) and the maps with them.
Keep `cmd/testtiming -redundancy` for an occasional sweep, so part of `internal/testcover` stays.
The red-first gate stays too: it uses `internal/testreach` without any map.
The mutation step is the closest call.

## Findings behind it

- 88 code-review findings in 55 runs used one of the three reports; none of the reports found a production defect on its own.
- Reach turns: 66, of which 23 added test functions.
- 22 of 24 redundancy findings kept the flagged test and marked it.
- Outside build and study tasks the manual verbs ran 6 times, and no test was removed on the word of `lym gate redundant`.
- The report steps took 7.0 hours of work and 2.8 hours of slot waiting over the window measured.
- The maps and their users are 11,363 non-test and 10,280 test lines.
- The effect of turning the steps off cannot be measured yet: no reviewed darn run has landed with all of them off.
  The report records the baseline and the script that gives the comparison later.

## Merge danger

**Door:** two-way
**Blast radius:** docs
