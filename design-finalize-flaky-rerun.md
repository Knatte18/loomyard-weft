# Finalize spawns a fixer on a single non-repeating test failure

`finalize-flaky-rerun` · Task · enhancement, landing, gate

Depends on: [`gate-target-dropped-silently`](design-gate-target-dropped-silently.md)

Imported from [issue #100](https://github.com/Knatte18/loomyard/issues/100).

## What was seen

A beam's Finalize landing ran the gate (tier0 and tier1) on the beam head.
One tier0 test of 3463 failed with `UnauthorizedAccessException`, and tier1 passed.
The test enumerates the source tree, `bin/` included, while another test running in parallel held a temporary directory under `bin/` with its read permission removed.
The failure is a race between two tests of the repository, not a fault of the change being landed.

Finalize wrote `.lym/verify/landing-failure.yaml` and spawned a fixer strand at once, on that single run.
The gate's own failure report said that no failing test or error line was read from the tier0 output, so the fixer started without a named cause.

## Workaround

The orch read the verify log, diagnosed the race and messaged the fixer with the cause.

## Expected behaviour

- Before Finalize spawns a fixer, it reruns the failed tests (or the failed tier) once on the same head.
  A failure that does not repeat is recorded as flaky in the landing report, and the landing goes on.
- The gate's failure report names the failing test and its error message when the test runner printed them, instead of reporting that no failing test line was read.
