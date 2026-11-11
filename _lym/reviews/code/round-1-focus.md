---
round: 1
focus:
  - "internal/gatecli/testmapyield.go and internal/gateslot/pool.go: does the spare take succeed only while two slots are free and no drain waits, and does the yield kill runs, write finished entries, and release both the slot and the build lock?"
  - "internal/gatecli/testmappackage.go: after a retake, does the pass retry the build lock without waiting, skip the package on a held lock, and re-read and re-plan the map at H before running another test?"
  - "internal/gatecli/testmap.go: under --upkeep, does the verb refuse package arguments, register as RoleTestMap, refuse beside a live pass, and exit before planning on a failed registration; is the exit record read and written only under --upkeep?"
  - "internal/gatecli/testmap.go and testmapplan.go: is HEAD read before each job and each wake, and does queued work leave out dirty-skipped, failed-at-HEAD and spawn-marked work?"
  - "internal/gatecli/testmappackage.go and internal/testcover/testfuncs.go: does the side-by-side detection accept only a top-level t.Parallel() preceded by nothing but tiergate.Require, is concurrency capped at the slot's tier-0 parallel, and does each run use its own profile?"
  - "internal/gatecli/testmappackage.go: after each round, are HEAD and the linked-package dirty read checked, and is a moved round discarded without writing, keeping earlier rounds; does a terminating signal write finished rounds?"
  - "internal/gatecli/testmaporder_test.go and testmap.go: is the job order by summed per-package elapsed seconds from gateslot.RunRecord, largest first, then import path?"
  - "Hand calls: do calls naming packages keep the blocking take, wait on the build lock, never yield, and rebuild whole; does a call under an inherited slot neither take nor yield; does a no-package call without --upkeep leave the exit record untouched?"
  - "Wake and wait: does the slot wait wake at most once a second with backoff as pattern/PATTERN-daemon-wakeups.md now says, and does a hub with fewer than two slots plan, re-stamp and end without taking one?"
  - "Test economy: the new tests in internal/gatecli, internal/gateslot and internal/testcover against the redundancy report, and any behavior the diff adds with no coverage."
---

Seed focus drawn from decision record D8 to D10 and the acceptance criteria, against the changed files in the review range.
