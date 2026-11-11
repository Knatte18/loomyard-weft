---
round: 2
focus:
  - "internal/gatecli/testmap_integration_test.go: does a row cover a head move with a non-body change to a linked file on a map that already holds entries, asserting the first written round leaves no entry whose Commit is the old one?"
  - "internal/gatecli/testmappackage.go (runTest, runTests, fillMapTests): does a test pin each outcome entry (failed, skipped, no coverage, timed out) and the rule that a run ended by stop or signal records no entry and no failure?"
  - "internal/gatecli/testmappackage.go (selfCheck) and internal/testcover/blockkey.go (ProfiledFuncs): does a test fail if the linked-function half of the self-check breaks, and does the self-check still run when no queued test wrote a profile?"
  - "internal/gatecli/redundancy_integration_test.go and mappedPackages: is a map holding entries beside a Failure field judged by its entries, and is a spawn-marked entry never counted as a cover, each with a test row?"
  - "internal/testcover/upkeep.go (PlanUpkeep): does the plan drop the entry of a test no longer declared in a mapped tier, and clear a Failure naming an older commit when the map moves to the new commit?"
  - "internal/gatecli/testmapaudit.go: does the audit's oldest-first order treat an entry with no MeasuredAt as oldest, and are the audit's guards (work queued, equal re-measure, no free slot) covered?"
---

Each entry names a site and the question to settle; the reviewer judges the code as the diff leaves it.
