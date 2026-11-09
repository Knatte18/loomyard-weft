---
round: 2
focus:
  - "In internal/gatecli/redundancyreport.go, settle whether retries() reruns the step after a call cancelled mid-run, whose unrun tests are listed as not judged with the cancelled reason."
  - "In internal/gatecli/redundancy_integration_test.go, settle whether deleting the FuncHash comparison in judgeRuns (internal/gatecli/redundancy.go) fails a test, using a map test not in the diff."
  - "Settle whether a test reaches the redundancy step through runTest or RunGate with a gate key listing reports, asserting the exit code stays the tiers' own and redundancy.json is written."
  - "In internal/gatecli/redundancy.go, settle whether a failed lym build and a corrupt coverage map (block index outside its table) each have a test asserting their report lines and that the gate result is unchanged."
---

Lower-severity coverage gaps (spawn-scan drops, mid-package budget, retry on an error-only report, match false cases, failed report write, slot hold) remain worth a look in the same files.
