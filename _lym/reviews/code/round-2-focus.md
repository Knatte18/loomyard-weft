---
round: 2
focus:
  - "internal/gatecli/memoryguard.go, memoryGuard.kills and otherHolders, with their callers in tierrun.go, copies.go, run.go and reports.go: does the run's own share count every process of the calling gate call, including sibling --copies copies and report-step test binaries, so that a call filling the memory slice itself is not classed as other load? Which memory charged to the slice does the share still miss, and does _lym/landing/summary.md name that bound?"
  - "internal/gatecli/tierrun.go, the slicePressure condition in runTierCommands and testFailures: can a build or vet failure, a TestMain exit or an init panic in a package other than the killed one still be classed as a stop that judged no code?"
  - "internal/verifytree/verify.go and the readers of its runner-crash flag (loomshed, landingshed, websterengine, webstercli, weaverengine), against gateslot.FindMemoryPressureStop and internal/gateslot/doc.go: does every verify site treat a memory pressure stop as not judged, with no fixer, as the board entry asks?"
  - "Tests for the memory pressure stop (internal/gatecli test_integration_test.go, copies_test.go, tierrun_test.go, memoryguard_test.go): would a test fail if the closing line, its precedence over the runner-crash line, the --stop-on-fail pass-through, the --copies fold or the mixed-stop exclusions were broken?"
  - "_lym/landing/summary.md against the board entry's ask to check the memory use of TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain: is the use backed by a measured figure from the current code?"
---

The entries cover the areas where the new pressure-stop behaviour changes next and needs a fresh read.
