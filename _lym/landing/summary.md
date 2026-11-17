# Report the mutation-confirmed redundancy candidates of every package

Adds `docs/research/speed/redundancy-confirmed-survey-2026-11-16/`: a report and its raw data.
It measures what `lym gate redundant` lists since `e97e2474` confirmed each candidate by mutation, across the repository.
No production code, test or Bolt file changes.

## What the survey found

- 430 tests stay confirmed across 94 of 121 judged packages, 3.45 s standalone together.
  The mutation check refuted 542 more candidates.
- 27 of 30 confirmed candidates read are false confirmations: each asserts something its covering tests do not.
  The kinds the mutator missed are absent side effects, persisted state, inputs the covering tests never feed,
  state kept across calls, pass-through values, error text and another package's result.
- Finding the 430 cost 9,252 s of mutation test runs.
  `internal/seatengine` is the worst return, 1,372 s for one confirmed test.
- `redundancy-sweep-existing` should read every confirmed candidate before it cuts, and the cut is not a speed task.

## Design decisions

- The package set is the earlier survey's candidate list plus three packages the first pass ran, not rebuilt from the new maps.
- The first 14 packages ran one at a time, and the other 102 ran three at a time, so their wall times are upper bounds.
- The 30-test sample is spread over the ten packages with the most confirmed tests, one subagent reading per test and no second check.
- Mutant test runs left untracked files in five source directories; they were deleted and are not committed.
