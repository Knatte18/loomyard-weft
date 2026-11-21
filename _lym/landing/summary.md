# Verify triage keeps first-run output and names command failures

The verify gate reruns a failing plan-level verify once, and records what failed the first run but passed on the rerun.
That record named only identities, and it listed a first run whose output named no failing test or package as a flaky test called `verify-command`.
So a failure of the verify step itself was filed as test flakiness, and the friction note kept no output to find its cause.

Now the `internal/websterengine` triage keeps each flaky failure's first-run log path and output tail.
A first run that named no failing test or package is reported as a failure of the verify command, apart from the flaky tests.
It is never reported as a flaky test.

Closes #160

Issue #158 (the flaky `internal/reedcli` package) is not closed:
this change does not touch `internal/reedcli` or find its nondeterminism.
It gives the triage note the first-run output that issue asked for, so the next flaky run can be diagnosed.

## Design decisions

- The failure kind `FailureKindOpaque` marks a first run that named no test or package, and it decides the split.
  `splitFlaky` divides the recorded failures into test and package ids and verify command logs.
- `Apply` writes both into `summary.md` through `AppendIntegrationTriage`, which now takes the command logs as a second list,
  and into the `webster-verify-triage` friction note, which keeps each failure's output tail in a fenced block.
- The fence is one backtick longer than the longest backtick run in the tail, so a fence line in the output cannot close the block.
- A command failure with an empty tail says its output was empty.
- The warning and the `logger.Warn` line name command failures by their first-run log.
- The spec docs (`webster-spec.md`, `loom-final-summary-spec.md`) and the describe stencil name verify command failures by log.
- `VerifyGateNotes.Flaky` is replaced by the unexported `flakyFailures`, which carries the failures with their logs.
  No caller of `Flaky()` remains in the repository.

## To check by hand

- The red-first gate was waived for this run: the diff changes `contracts/specs/webster-spec.md`, which the red-first overlay cannot restore on the merge base.
  `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` reaches the bug through base code (`newVerifyGate` and `VerifyGateNotes.Apply`).
- A friction note holding only a verify command failure, with no flaky test, has no test of its header text (`writeTriageFrictionNote` in `triage.go`).
- Nothing reproduced the `internal/reedcli` failure; run `lym gate test ./internal/reedcli` repeatedly if #158 matters to this review.

## Merge danger

**Door:** two-way
**Blast radius:** websterengine
