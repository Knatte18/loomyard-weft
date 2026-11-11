# Finalize reruns failed gate tests once before spawning a fixer

Finalize's landing gate used to spawn a fixer on the first failing run.
A test that failed on a race with another test therefore started a fixer for a fault the change did not cause.
The gate's failure report also named no cause for a VSTest failure.

Closes #100

## What changes

- `internal/landingshed`: when Finalize's gate fails, it runs the failure once more on the same head before it judges it.
  - It reruns only when the failure report lists tier and guard runs alone.
    A timeout, a runner crash, a build, vet or lint failure, and a package that failed outside any test are judged as before.
  - The rerun runs the failed tests alone when the report names Go tests only, and the whole gate call otherwise.
  - A rerun that passes makes the failure flaky: the gate passes and no fixer runs.
  - A rerun that fails is the gate's failure, and the landing failure record names the rerun's log.
  - The landing commit and the GitHub merge body list each flaky failure under a new `## Flaky test failures` section,
    after the regression check.
    The section is absent when there was none.
    `contracts/specs/loom-final-summary-spec.md` records it.
- `internal/gatecli`: the VSTest counter now reads each failed test's block.
  The failure report names the test and keeps the first lines of its error message and stack trace,
  instead of reporting that no failing test line was read.
- `internal/gateslot`: `TierRunFailures` reads the failure report's failures for the rerun decision.
  The report's fixed phrases are now exported constants shared by the producer and the parser.

## Design decisions

- The rerun is part of the gate's `check`, enabled by `onFlakyPass`, which Finalize alone sets.
  The fixer turns' gate reruns the same way; Parent-Align's gate runs no rerun.
- Flaky failures are held in the Finalize process.
  A Finalize resumed after a flaky pass lists none found before the resume.
- The decision to rerun reads the gate's failure report, not a runner's raw output,
  so it cannot judge a failure the report does not name.

## To check by hand

- No regression check ran, and nothing was run beyond the unit tests: try a real landing whose tier0 has a test failing once.
  It should pass with the `## Flaky test failures` section in the landing commit.
- Check that a VSTest failure's report shows the test name and its error message on a real .NET gate run.
- `newfinalize-flaky-sink-wiring-untested` (`internal/landingshed/finalize.go`, `NewFinalize` and `assembleFinalize`):
  no test covers that `NewFinalize` wires the gate's flaky sink to the Finalize it returns.
- `rerun-guards-and-outcomes-lack-rows` (`internal/landingshed/verifygate.go`, `rerunTestFailure`):
  the guards that skip a rerun and the rerun's outcomes lack test rows each.
