---
round: 2
focus:
  - "internal/battencli/lifecycle_integration_test.go: does any step that runs under a t.Parallel test or subtest touch process-global state, in particular stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer, which redirects the process-global logger through logcapture.Capture (PATTERN-test-speed, 'Tests run in parallel')? Where a step was moved to run serially, does it keep every assertion and build its own precondition, and does a comment name the global state?"
  - "internal/battencli/lifecycle_integration_test.go, the doc comment of TestBattenIntegration_Rows: does its rule on the order of steps that leave the hub changed match the table, given stepSeedChild_IgnoresPrimesCommittedBattenRecords, stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent and stepCreateRow_LeftoverBranchIsRewordedForPrime each leave the hub changed?"
  - "_lym/landing/summary.md, '## Measurements': does it name the exact command and tier of its runs, and the set-up difference from the 33 to 75 s figure of docs/research/speed/test-time-measure-2026-11-27/report.md (its 'Machine and load' section), and does every figure and claim there hold for the test layout the current commits leave?"
---

The last entry covers figures that a change to the test layout can make obsolete.
