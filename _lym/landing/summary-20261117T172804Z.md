# gate redundant: timeouts are no kills; per-test seconds to the µs

`lym gate redundant` counted every mutant run that ended at its time bound or at the memory cap as a kill.
Such a run proves no check of the mutant, so it could confirm a candidate whose covering tests only stalled on the same mutant.
The map-tooling trial on `internal/boardengine` found 383 timed-out and 39 memory-capped runs among 30994, all counted as kills.
The same trial found that 132 of 136 tests in the coverage map recorded `seconds: 0`, so the greedy "slowest first" pass had no order to go by.

## Unjudged runs and unconfirmed candidates

- `testcover.JudgeMutants` takes the unjudged runs, in the shape of the kills, beside the kills.
  An unjudged run counts as no kill.
  A candidate's answer rests on an unjudged run when its own run of a mutant ended that way,
  or when it killed a mutant that no covering test killed and some covering test's run of it ended that way.
  `MutantVerdict.Unjudged` lists those mutants.
- A candidate with a unique kill is still refuted.
  One without a unique kill whose answer rests on an unjudged run is neither redundant nor refuted.
  `lym gate redundant` lists it under a new `unconfirmed` key with its tier, seconds, covering tests, `mutants_killed` and `unjudged_mutants`.
  A candidate that killed nothing and had no unjudged run is still not judged with `kills no mutant of its blocks`.
- `mutation.timed_out` and `mutation.memory_capped` still list every such run with its mutant and test.
- In `runMutants`, a covering test that is itself a candidate and timed out on a mutant beside other runs is queued again for a run alone,
  the same as a candidate run that killed the mutant.
  A memory-capped run is not run again: its cap applies to that run, so it would hit the cap alone as well.
- In the `TestGateRedundant` fixture, the loop tests whose only kill of the loop-ending mutant came from a timeout or the cap move from `candidates` to `unconfirmed`.
  The run count drops by the covering runs of those mutants, which no longer start.

## Per-test seconds

- A test binary prints each test's time rounded to the hundredth of a second.
  `Toolchain.RunAlone` now stamps when the test's `=== RUN` line arrives and when its result line arrives, and records the difference to the microsecond.
  The binary's own figure stays only for output with no `=== RUN` line.
- `Toolchain.Now` is the clock behind the stamps, nil for `time.Now`.
  The new runner row uses a stepping clock, so the expected 0.0015 s holds however the output is chunked.
- The map's `seconds`, the mutant timeout and the report's seconds all read `RunResult.Elapsed`, so they all get the finer figure.

## Design decisions

- The `funcs` order item ("was B4") has no code change.
  `fillMapTests` writes each entry's `blocks` and `funcs` as sorted index lists: a new run's through `slices.Sort` and `slices.Sorted`, a kept entry's through `reindexed`.
  The function table is sorted by `CompareFuncKeys`, which orders every pair of distinct keys, and `state.WriteJSON` reorders nothing.
  Both sorts predate `b92c95ee`, the commit the trial ran at, and a map in the hub's store holds sorted `funcs` for every test.
  With equal sets on both sides, the reported order difference cannot come from the code at that commit.
  A `lym` binary older than the commit the trial ran at would explain it.
  If a later map build shows it again, a byte comparison of two builds on one commit would locate the cause.
- Unconfirmed is a separate list rather than a reason under `not_judged`.
  The mutants it carries tell the operator which runs to inspect, which `not_judged` has no field for.
