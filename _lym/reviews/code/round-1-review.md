---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/gatecli/testmappackage.go (fillMapTests), internal/gatecli/testmapaudit.go:21
    summary: the audit replaces every re-measured entry, equal ones too, while D7 replaces only on a differing result
    fix: Leave the code in place and defer to the run's parent, who either records the rotation rule with lym loom decision add (an audit re-measure refreshes the entry's MeasuredAt whatever the result) or refuses it, in which case audit rotation needs a separate marker so an equal result leaves the kept entry untouched.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:code-affected-set
    location: internal/gatecli/testmap.go:228-247, internal/dobbyengine/testmap.go
    summary: D12 acceptance says the dobby reads the exit record, and no dobby code reads it
    fix: Leave the code in place and defer to the run's parent. The change description defers the read to the dobby entry of D8; the parent records that with lym loom decision add, or refuses it and puts the read here.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-affected-set
    location: internal/dobbyengine/config.go:64-65, internal/dobbyengine/template.yaml:4-5
    summary: the config-layers constraint retires test_map_batch and test_map_every as keys of dobbyengine and the diff retires neither
    fix: Leave the code in place and defer to the run's parent. Retiring the keys here would break the landing-count start of the refresh, so the parent records with lym loom decision add that the retirement belongs to D8's entry.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go (HeadMoveRetriesTheFailedBuild subtests, about line 683)
    summary: no test covers a non-body miss, or any miss on a map that holds entries, so the whole rebuild's empty start has no row
    fix: Add a subtest row to the existing testmap integration test whose map holds entries measured before HEAD and whose HEAD makes a non-body change to a linked file; assert that after the first written round the map at HEAD holds no entry whose Commit is the old one.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundancy_integration_test.go:170-176
    summary: no row covers a map that holds entries beside a Failure field, nor a spawn-marked entry that would otherwise be a cover
    fix: Add a fixture row to TestRedundancyStep with a map holding entries plus a Failure field and assert its tests are judged by the entries; add a spawn-marked entry sharing blocks with a judged test and assert it is not counted as a cover.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmappackage.go (runTest, runTests, fillMapTests)
    summary: only the timed-out outcome entry is asserted; the failed, skipped and no-coverage entries and the nothing-recorded rule for an ended run have no row
    fix: Add rows to the testmap integration test for a failing test, a skipping test and a test covering nothing, each asserting its OutcomeFailed, OutcomeSkipped or OutcomeNoCoverage entry with no coverage; add a cancelled-context row asserting no entry and no failure is written.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmappackage.go (selfCheck lostFunc branch), internal/testcover/blockkey.go (ProfiledFuncs)
    summary: the self-check's linked-function half has no test; only the block-key half is pinned
    fix: Add a subtest to the testmap integration test in which a kept entry names a linked function the fresh build lacks, and assert that entry is re-measured and counted in self_check_mismatches.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/upkeep.go (loop over in.Map.Tests)
    summary: PlanUpkeep keeps the entry of a test no longer declared in a mapped tier, so it survives every Current verdict
    fix: In PlanUpkeep drop an entry whose name is not in in.Declared and set Changed; add a TestPlanUpkeep row for a removed test.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/upkeep.go (m := in.Map)
    summary: a plan that moves a map to H keeps a Failure naming an older commit, which no later step clears when nothing is queued
    fix: In PlanUpkeep set m.Failure to nil and Changed to true when in.Map.Failure is set and its Commit differs from in.Commit; add a TestPlanUpkeep row.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/testmappackage.go (self-check guard on firstProfile, r.measures append)
    summary: the self-check is skipped when no run wrote a profile, and r.measures is appended before the dirty and changed-under-tests skips
    fix: Run the self-check from a fresh block table that does not depend on a test's profile, or count and report the skipped check; move the r.measures append after the skip checks so a skipped package reports no build or run seconds.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/testmapaudit.go:42-45
    summary: an entry with no MeasuredAt takes the map's BuiltAt, which every re-stamp resets, so legacy entries look young and the audit's oldest-first order is wrong
    fix: Treat a zero MeasuredAt as the zero time in the audit's ordering, so such entries sort oldest, and drop the BuiltAt fallback and its comment.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/testmap.go:13, internal/dobbyengine/config.go:42, internal/dobbyengine/template.yaml:5
    summary: three dobby texts still say the refresh builds at most test_map_batch stale packages
    fix: Reword the three to the wording the diff gave dobbyengine/doc.go, that it plans every map at the prime's HEAD and measures at most test_map_batch of the packages it queues.
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmapaudit.go, internal/gatecli/testmap.go (runTestMap envelope)
    summary: the audit's guards and the verb's plan and packages output have no row
    fix: Extend the audit subtest of the testmap integration test with a pass that leaves work queued (no audit), an equal re-measure (no mismatch) and no free slot (no audit); decode plan and packages in testMapOutcome and assert them non-empty after a measuring call.
  - id: F14
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmapplan.go:70-73,82-84, internal/gatecli/testmappackage.go (planned.Commit != head skip)
    summary: the plan's two fall-back-to-whole-rebuild paths, the stale planned map skip, and the plan step holding no gate slot have no row
    fix: Add rows to the testmap integration test for a mapped-test scan error queueing a whole rebuild, a map rewritten past the plan being skipped and counted queued, and a plan-only pass leaving the slot pool free.
---

### [MEDIUM] The audit replaces equal entries, which D7 does not say
**Location:** internal/gatecli/testmappackage.go (fillMapTests), internal/gatecli/testmapaudit.go:21
**Issue:** D7 says a fresh audit result that differs from the kept one replaces it and is counted.
The audit job runs `fillMapTests` for every re-measured test, so an equal result replaces the kept entry too and moves its `MeasuredAt`.
The change description names this as the way the next round reaches other entries; the decision record does not carry it.
The rotation is sound, and the open choice is whether D7 says so.

### [MEDIUM] No dobby read of the exit record
**Location:** internal/gatecli/testmap.go:228-247, internal/dobbyengine/testmap.go
**Issue:** The D12 acceptance criterion ends "and the dobby reads it".
`ReadExitRecord` has no production caller, and the change description says so and defers it to D8.

### [MEDIUM] test_map_batch and test_map_every stay live
**Location:** internal/dobbyengine/config.go:64-65, internal/dobbyengine/template.yaml:4-5
**Issue:** The record's `PATTERN-config-layers` constraint has these two keys leave their templates as retired keys.
The diff changes only the `test_map_max_age_days` key, in gateslot.
The record's scope lists no dobbyengine package, and the dobby still starts the refresh by landing count, so the retirement reads as D8's.

### [MEDIUM] A miss on a map with entries is untested
**Location:** internal/gatecli/testmap_integration_test.go (HeadMoveRetriesTheFailedBuild subtests, about line 683)
**Issue:** The two misses in the integration run are `MissBuildFailure` and `MissNoMap`, neither of which has earlier entries to drop.
The D1 criterion that the map after a rebuild's first round holds no entry measured before HEAD has no row, and seeding `m` from the old map in the `job.whole` branch of `mapPackage` would break no test.

### [MEDIUM] Redundancy cover rules lack rows
**Location:** internal/gatecli/redundancy_integration_test.go:170-176
**Issue:** The `broken` fixture is still a `BuildFailure`-only map, so the changed `mappedPackages` rule that judges a map with entries and a `Failure` field by its entries has no row.
The D4 rule that a spawn-marked entry is no cover has no row either; only the failed-outcome row pins `Measured()`.

### [MEDIUM] Outcome entries other than timed-out are unpinned
**Location:** internal/gatecli/testmappackage.go (runTest, runTests, fillMapTests)
**Issue:** `CapLeavesTheRestQueued` asserts the `OutcomeTimedOut` entry alone.
The failed, skipped and no-coverage entries, and the D4 rule that a run ended by stop or signal records nothing, are code the diff adds with no row.

### [MEDIUM] Self-check of linked functions is unpinned
**Location:** internal/gatecli/testmappackage.go (selfCheck lostFunc branch), internal/testcover/blockkey.go (ProfiledFuncs)
**Issue:** D7 asks the self-check to check own-package block keys and linked functions.
The one self-check subtest drives the block-key half, so `ProfiledFuncs` and the `lostFunc` branch run in no test that would fail if they broke.

### [LOW] A removed test keeps its entry
**Location:** internal/testcover/upkeep.go (loop over in.Map.Tests)
**Issue:** The loop drops an entry only for an affected test or an outcome entry after an own-file change.
`in.Declared` serves the queue alone, so the entry of a test deleted or re-tiered since the map's commit survives every Current verdict, where the old whole rebuild dropped it.
The selector's `own.tests` was not shown to report removed tests, so the effect on gate selection is unverified; the stale entry still reaches the redundancy step.

### [LOW] A stale Failure survives a re-stamp
**Location:** internal/testcover/upkeep.go (m := in.Map)
**Issue:** `m := in.Map` carries `Failure` through a Current or Affected plan.
Only a measuring job clears it, so a Current map with an empty queue holds a failure at an old commit indefinitely.

### [LOW] Self-check skipped without a profile; measures recorded before skips
**Location:** internal/gatecli/testmappackage.go (self-check guard on firstProfile, r.measures append)
**Issue:** A non-whole job whose every queued test fails, times out or skips writes no profile, so the kept entries go unchecked against the fresh build on a build D7 says is always checked.
`r.measures` is also appended before the dirty and changed-under-tests skips, so a skipped package still reports seconds and a test count.

### [LOW] Legacy entries age from the map's BuiltAt
**Location:** internal/gatecli/testmapaudit.go:42-45
**Issue:** `writeMap` sets `BuiltAt` to the time of every write, so each re-stamp makes entries with no `MeasuredAt` look newer.
Oldest-first then ranks them behind entries measured later than they were.

### [LOW] Stale refresh wording in dobbyengine
**Location:** internal/dobbyengine/testmap.go:13, internal/dobbyengine/config.go:42, internal/dobbyengine/template.yaml:5
**Issue:** The diff reworded `dobbyengine/doc.go` and left these three saying the refresh builds stale packages.

### [LOW] Audit guards and verb output unasserted
**Location:** internal/gatecli/testmapaudit.go, internal/gatecli/testmap.go (runTestMap envelope)
**Issue:** `runTestMapIn` sets `auditTests` to 0 outside the one audit subtest.
No row covers the audit being skipped while work is queued, an equal re-measure not counting, or no free slot.
`testMapOutcome` decodes neither the `plan` nor the `packages` key the verb now prints.

### [LOW] Plan fallbacks and the no-slot plan step unpinned
**Location:** internal/gatecli/testmapplan.go:70-73,82-84, internal/gatecli/testmappackage.go (planned.Commit != head skip)
**Issue:** The two fall-back-to-whole-rebuild paths in `planPackage`, the non-whole job skipped when its map no longer stands at HEAD, and the PATTERN-gate-slot rule that the plan holds no slot have no row.

## Rejected

- Environment stamp mismatch between `headSelector` and `writeMap`: the selector adds go.mod and go.sum through `StampModuleFiles` itself (`testselect.go:246`), so the stamps agree.
- Plan fallbacks not counting a verdict or logging the swallowed error: the whole rebuild they queue records the failure, so the error reaches the map and the report.
- No `Pattern-Departure:` trailer: the diff departs from no PATTERN entry, and the constraints declare no PATTERN add, change or remove.
- Redundancy-report candidates: every listed test reads "not judged", so no test-economy finding rests on it.
