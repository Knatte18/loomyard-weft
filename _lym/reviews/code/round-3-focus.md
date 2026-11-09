---
round: 3
focus:
  - "In `redundancyRun.runTest` (internal/gatecli/redundancy.go), settle whether a test killed in flight when the step's context is cancelled is listed as not judged with `reasonCancelled`, so `retries()` reruns the step on the next call."
  - "In internal/gatecli/redundancy_integration_test.go `TestRedundancyStep`, settle whether a test cancels the step's context mid-run and asserts both the unrun tests and the in-flight test carry `reasonCancelled` and `report.retries()` is true."
  - "In `judgeRuns` (internal/gatecli/redundancy.go), settle whether a test pins the `judged[mapped.Name]` exclusion: a judged test whose map entry `FuncHash` equals its current hash must not be its own cover."
  - "In internal/gatecli/redundancy_integration_test.go, settle whether a test asserts the step holds a gate slot with Site `redundancySite` while it runs and releases it after, per `PATTERN-gate-slot`."
---

Round 2's unresolved items are one cancel-path defect and three test gaps; the focus names where each is settled.
