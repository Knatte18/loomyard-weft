# Round 5 fixer report

All six findings are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged.

## Changes

- F1: Decision 13 gains a bound that a family merges only among tests of one tier and a merge never raises a tier.
- F2: Decision 13's independence bound defines the fixture's state as everything a later step can observe, bars a step that cannot return all of it, says a comment naming a mutation does not make a step eligible, and requires a restoring step to assert the restored state.
- F3: Decision 13's timing bound names the package elapsed time `go test` reports as the measure, fixes the order of before runs, merge and after runs, discards a run whose load differs by more than 25% from the first before run, defines the spread as the range of the three before runs, and reverts the merge on a tie or a lower after median within that spread.
- F4: Decision 8's command gains `-count=1`, and a run that reports no tests to run is not a verdict.
- F5: Decision 13 says Decision 4's per-subtest `t.Parallel` applies to folded tables only and that a protected test stays its own top-level test, never a step; Decision 2 says a fold may host a receiving test and a merge never does.
- F6: Decision 10's last reason class reads "no fold or merge applies to", and the Goal lists the scenario merge.

## Deferred

None.

## Notes

The 25% load tolerance in F3 is a choice the review left to the author; the record can change it without touching the rest.
