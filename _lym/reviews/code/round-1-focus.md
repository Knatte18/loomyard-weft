---
round: 1
focus:
  - "internal/gatecli/testmapplan.go, testmappackage.go: does the plan return Current, Affected and Miss as D1 defines them, and does Current re-stamp the map and marker (Commit, BuiltAt, environment stamp) without building or running anything?"
  - "internal/gatecli/testmapplan.go: does the plan queue declared mapped tests with no entry under every verdict but Miss, and does a Miss leave the map untouched and queue a whole rebuild that starts from an empty map?"
  - "internal/gatecli/testmap.go, testmappackage.go: is a map rewritten only under the package's build lock, with a held lock skipped and counted as queued work in the exit record; does the plan step hold no gate slot while every build and run does?"
  - "Outcome entries (D4): are failed, timed-out, skipped and no-coverage runs recorded with no coverage; are they kept on Current and dropped on a miss or a change to the package's own files; is nothing recorded for a run ended by yield, stop or signal; are spawn-marked tests recorded, never run and never queued?"
  - "D5 failed build: are earlier entries, Commit and marker kept, the failure and H recorded in the separate field, a failure-only map still written for a package with no entries, and the package not requeued at the failed HEAD? Check internal/testcover/coveragemap.go and the redundancy step's reading of a map with entries plus a failure field."
  - "D7: does the self-check compare kept entries' own-package block keys and linked functions with the fresh build on every build, and does the audit run once per pass after the queue is empty, only when a slot is free at once, with a named constant count, oldest measurement first, replacing only on a differing result (internal/gatecli/testmapaudit.go, internal/gateslot/pool.go TryAcquire)?"
  - "Retirement of test_map_max_age_days (gateslot) and the config-layers constraint: is the key left as a retired key so a hub base file holding it still loads, and are test_map_batch and test_map_every in dobbyengine handled as the constraints section says? Check whether the diff carries those changes."
  - "D12 exit record (internal/testcover/exitrecord.go, gatecli report): are the planned HEAD, queued count, verdict counts with miss reasons, mismatch counts and per-package build and run seconds and test counts present, tolerant to old records, and printed by the verb; is Unbuilt replaced and every reader updated, including the dobby?"
  - "internal/testcover/upkeep.go and blockkey.go: correctness of the pure plan logic against the selector's target and stamp semantics, and CodeFileAt as the only way file content at Commit and H is read."
  - "Test economy: TestPlanUpkeep, TestTryAcquire_TakesAFreeSlotAndNeverWaits, the map tests and the testmap integration tests; do they cover the new behaviours (self-check, audit, D5 field, outcome entries, redundancy cover rule) without duplicating one another, and is any behaviour added without coverage?"
  - "Change description and commit messages: does _lym/landing/summary.md, when present, claim anything the diff does not bear out; are doc.go and PATTERN-test-speed.md updates consistent with the new behaviour?"
---

Seed focus from the decision record's D1, D4, D5, D7 and D12 and its constraints.
