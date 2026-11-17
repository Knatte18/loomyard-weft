# Count timed-out and memory-capped mutant runs as unjudged

`lym gate redundant` counted a mutant run that ended at its time bound or at the memory cap as a kill.
Such a run proves no test checked the mutant, so it inflated the kills of the tests around a candidate and made candidates look redundant when they were not.
The map-tooling trial on `internal/boardengine` saw 383 timeouts and 39 memory-cap ends among 30994 mutant runs, all counted as kills.

This change makes those runs unjudged and reports the candidates that rest on them apart.
It also records each test's seconds at a resolution that separates tests.

## What changes

- A mutant run that ends at the time bound or the memory cap is no kill.
  `mutation.timed_out` and `mutation.memory_capped` still list those runs.
- `lym gate redundant` prints a new `unconfirmed` list beside `candidates` and `refuted`.
  An entry holds the test's tier, seconds, covering tests, `mutants_killed` and `unjudged_mutants` (file, line, operator).
  A test is unconfirmed when it has no unique kill and its answer rests on an unjudged run:
  its own run of a mutant none of its covering tests killed,
  or its covering tests' runs of a mutant it killed when none of them killed it.
- A unique kill still refutes a test, even when it also has unjudged answers.
- A candidate's own unjudged run of a mutant a covering test killed is moot.
  Deleting the candidate loses no check of that mutant, so the run is in none of the candidate's lists.
- A candidate whose own run was unjudged now has its covering tests run that mutant, as a killing candidate already did.
- A covering test that times out beside other runs runs once more alone, and that run stands, as a kill already did.
  A covering test that timed out as a candidate runs again for the same reason.
- Per-test seconds in the coverage map run from the arrival of the test's `=== RUN` line to that of its result line, to the microsecond.
  The test binary rounds its own figure to the hundredth, and most tests recorded 0.
  A run with no `=== RUN` line before its result line keeps the binary's figure.
  This gives the greedy pass of `lym gate redundant` an order to work from.
- The `redundancy_test_timeout_sec` and `redundancy_test_memory_mb` comments in `internal/gateslot/template.yaml`, the `internal/gateslot` and `internal/gatecli` docs, `docs/overview.md` and `pattern/PATTERN-test-speed.md` now say unjudged where they said kill.

## Design decisions

- `testcover.JudgeMutants` takes a second `Kills`-shaped argument, `unjudged`, and `MutantVerdict` gains `Unjudged`.
  `Redundant` now also requires that no answer rests on an unjudged run.
- A candidate with no kill and no unjudged answer is still not judged.
  One with only unjudged answers is unconfirmed rather than not judged, so the operator sees the mutants to look at.
- The timing is stamped by a `testOutput` writer in `internal/testcover/runner.go`, with an injectable `Toolchain.Now` clock for tests.
  The stamps cover wall time between two lines, so a loaded machine inflates a figure.
  The report still reads as a floor for what deleting a test saves.

## Not in this change

The sort of each test's `funcs` and `blocks` before the map is written, the other defect the trial found, is not in this diff.
Check whether the map still rewrites its `funcs` arrays in a different order between builds on one commit.

## To check by hand

- Run `lym gate redundant internal/boardengine` against a fresh map.
  Confirm that `unconfirmed` is populated, that the candidates' `seconds` are no longer all 0, and that the greedy pass has an order.
  This was not run on a real package.
- `own-unjudged-run-makes-candidate-unconfirmed` (`internal/testcover/mutantverdict.go`, `internal/gatecli/redundantmutants.go`):
  check that a candidate's own unjudged run of a mutant makes it unconfirmed, as intended, and that the moot case above holds.
- `timed-out-covering-rerun-untested` (`internal/gatecli/redundantmutants_guard_test.go`):
  check that the rerun of a covering test that timed out beside other runs is pinned.
  The new test covers a covering test that timed out as a candidate.
- `testoutput-line-assembly-untested` (`internal/testcover/runner.go`, `internal/testcover/stampoutput_test.go`):
  check that `testOutput` assembles a split `=== RUN` or result line and skips an indented subtest result line.
- `gate-template-comments-still-say-timeout-is-kill` (`internal/gateslot/template.yaml`) and `gateslot-doc-still-says-timeout-is-kill` (`internal/gateslot/doc.go`):
  check that no wording left says a timeout or memory cap is a kill.
