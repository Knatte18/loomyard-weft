---
round: 2
focus:
  - "internal/landingshed/finalize_gate_test.go and finalize.go: does a test fail when NewFinalize stops wiring the flaky-pass sink into the verify gate?"
  - "internal/landingshed/verifygate.go rerunTestFailure and verifygate_test.go: does each rerun guard (runner crash, negative exit code, empty log) and each rerun outcome (timeout, skipped, different failure) have a pinning row?"
  - "internal/gateslot/runlog.go TierRunFailures: is a package that failed outside any test, such as a compile error, treated consistently with the doc's claim that a build failure is not rerun?"
  - "internal/gatecli/failurereport.go and internal/gateslot: do the report producer and its parser share one definition of the output-line and cut-line strings?"
---

Round 2 re-checks the fixes from round 1 and the areas around them.
