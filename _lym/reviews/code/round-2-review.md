---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/gatecli/doc.go:1112
    summary: "The package doc says an entry with no measurement time counts from its map's build, but the audit now sorts such an entry oldest."
    fix: "In internal/gatecli/doc.go line 1112, replace the sentence `An entry with no measurement time counts from its map's build.` with `An entry with no recorded measurement time counts as the oldest.`, matching the audit doc comment in internal/gatecli/testmapaudit.go."
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/testmap_integration_test.go:1427
    summary: "No test pins that an audit candidate with no MeasuredAt sorts before dated ones; the ordering rule fixed in round 1 has no coverage."
    fix: "In the subtest TheAuditRunsOnlyWithNothingQueuedAndASlotFreeAndCountsOnlyADifferentResult, add a step before the final equal-result call. Write the caller map with TestCallerCalls MeasuredAt set to the year 2000 and TestCallerAdded MeasuredAt left zero and its Blocks cleared, run `call(t, environ, 10, 1)`, and assert AuditMismatches is 1 and TestCallerAdded is the entry re-measured, with its blocks back and MeasuredAt set. Fold it into that subtest as an assertion rather than a new test."
  - id: F3
    severity: LOW
    class: design
    location: internal/gatecli/testmappackage.go:110
    summary: "The cancel checks added after the go list, coverage build and lym build record no failure, but only a call cancelled mid-run is tested."
    fix: "Extend ACallEndedMidRunRecordsNoEntryAndNoFailure in internal/gatecli/testmap_integration_test.go into a table of two rows, one cancelled while a test hangs and one cancelled while the fake `go build` hangs. The build row needs a hang switch in the helper's go build role, like the one the test hang uses. Assert for both that runTestMap reports the cancelled call and that ReadMap finds no map and so no BuildFailure or Failure."
---

### [LOW] Package doc contradicts the audit's treatment of an entry with no measurement time

**Location:** internal/gatecli/doc.go:1112
**Issue:** The round-1 fix for F11 changed `audit` in `internal/gatecli/testmapaudit.go` to sort an entry whose `MeasuredAt` is zero as the oldest, and rewrote that function's doc comment to say so.
The package doc kept the earlier rule, "counts from its map's build".
The doc now describes behavior the code no longer has.
The change description does not state the rule either way.

### [LOW] The oldest-first rule for an entry with no MeasuredAt is untested

**Location:** internal/gatecli/testmap_integration_test.go:1427
**Issue:** The audit's order is `time.Time.Compare` over `MeasuredAt`, so a zero time sorts first only because the zero value is the earliest.
Both audit rows in `TestGateTestMapUpkeep` and `TestGateTestMapLinkedPackages` give the oldest entry a dated `MeasuredAt` of the year 2000, so no row runs an entry with a zero time against a dated one.
The focus asked whether the order treats such an entry as oldest. The code does, but a change to how the audit orders or substitutes the time would not fail any test.
Maps written before `MeasuredAt` existed are the real source of such entries.

### [LOW] A build ended by a terminating signal has no test

**Location:** internal/gatecli/testmappackage.go:110
**Issue:** D4 says a run or build ended by a terminating signal records no outcome entry and no failure.
`mapPackage` has three build-side branches for this, after `LinkedPackages`, after `BuildBinary` and inside `buildLym`, each returning the context's error before `writeFailed` runs.
`runTests` has the run-side rule, and `ACallEndedMidRunRecordsNoEntryAndNoFailure` pins it: a cancel while a test hangs leaves no map, so neither an entry nor a failure was written.
No test cancels during a build, so removing any of the three checks would write a `BuildFailure` map for a call the user stopped and still pass.
## Rejected

- Focus 1, the head move with a non-body change to a linked file on a map with entries.
  `AMissOnAMapWithEntriesRebuildsItFromNothing` plants a ghost entry measured at the old commit, makes a non-body change to the package's own file, and asserts the rebuilt map holds only `TestHeadmoveOne` and no entry whose `Commit` is not HEAD.
  The row uses the package's own file, not a linked one, but `PlanUpkeep` returns `UpkeepMiss` for any selector miss without reading its source, and `mapPackage` starts a whole job from an empty map.
  The linked case takes the same code, so a row for it would repeat this one. Not a defect.
- Focus 2, the outcome entries.
  `TestGateTestMap` pins the timed-out entry through `TestAlphaHangs`, and `EveryOutcomeGetsItsEntryWithNoCoverage...` pins failed, skipped and no-coverage.
  The cancelled-run rule is pinned apart from the build branches in F3.
- Focus 3, the linked-function half of the self-check and the no-profile case.
  `AKeptEntryNamingALinkedFunctionTheBuildLacksIsReMeasured` plants a function the build lacks and expects one self-check mismatch, which a broken `lostFunc` test would miss.
  `APassWhoseQueuedTestsLeaveNoProfileStillChecksTheKeptEntries` has the only queued test fail with no profile and still expects the mismatch, through `runFirstKeptEntry`.
  Not a defect.
- Focus 4, `mappedPackages` and the redundancy step.
  `betaMap.Failure` beside an entry is still judged, and the `drops` map's failed and spawn-marked entries cover nothing, so `TestDropsLate` stays unique.
  Neither test's name is among the runs, so `Measured` is what excludes them.
- Focus 5, `PlanUpkeep`.
  `ARemovedTestDropsItsEntry` and `AFailureAtAnEarlierCommitIsCleared` cover the two named rules.
- Focus 6, the audit guards.
  Work queued, an equal re-measure and no free slot are each asserted in the audit subtest.
- The redundancy report's tests listed as not judged are out of process or in packages with no coverage map. None is a test-economy candidate.
- The `dobbyengine` edits change wording only. The keys stay, as the record's added entries require.
- D1 to D7 and D12 are otherwise carried by the diff, and `test_map_max_age_days` is a retired key with a test that a hub file still setting it loads.
