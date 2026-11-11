```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

| Behaviour | Pinned by |
|---|---|
| Current verdict re-stamps the map and marker, builds nothing (D1) | `TestPlanUpkeep` `CurrentMovesTheMapToTheCommitWithEveryEntry` and `CurrentAtTheMapsOwnCommitChangesNothing`. The integration subtests `ALinkedCommentChangeMovesTheMapToHeadWithEveryEntry` and `HeadMoveRetriesTheFailedBuild…` check commit, marker and `Built` empty. |
| Affected verdict drops the affected tests' entries and re-measures only those (D1) | `TestPlanUpkeep` `AffectedDropsAndQueuesTheAffectedTests`, and the integration subtest `ALinkedBodyChangeReMeasuresOnlyTheTestsThatCoveredIt`. |
| Declared tests with no entry are queued; a spawn-marked test gets an entry and is never queued | `TestPlanUpkeep` `ADeclaredTestWithNoEntry…`. The linked fixture pins the spawn entries with no coverage. |
| Own-file change drops outcome entries; a linked change keeps them (D4) | `TestPlanUpkeep` `AnOwnFileChange…` and `ALinkedChangeKeepsTheOutcomeEntries`. The integration `TestAlphaHangs` entry appears after the first call and is dropped by the own-file edit. |
| Miss leaves the map untouched and queues a whole rebuild | `TestPlanUpkeep` `AMissLeavesTheMapAndQueuesNoTest` (plan level only). In integration, `HeadMove…` covers only the `MissBuildFailure` and `MissNoMap` reasons. |
| Whole rebuild starts from an empty map, so no entry measured before HEAD survives | None. Neither miss in the integration run has earlier entries to drop. The non-body miss and an old map with entries are untested. |
| Timed-out run records an outcome entry | `CapLeavesTheRestQueued` asserts `TestAlphaHangs` as `OutcomeTimedOut`. |
| Failed, skipped and no-coverage outcomes | None visible. Only the timed-out case is asserted. |
| Run ended by signal or stop records nothing (D4) | None. The context checks in `runTests` and `runTest` are unexercised. |
| `Measured()` and the redundancy step counting only passing entries (D4) | `TestRedundancyStep`, where a failed-outcome row leaves `TestDropsLate` unique. No row has a spawn-marked entry that would otherwise be a cover. |
| Redundancy step judges a map with entries and a `Failure` field by its entries (D5) | None. The `broken` fixture is still a `BuildFailure`-only map. |
| Failed build keeps entries, `Commit` and marker, records `Failure` with HEAD (D5) | The integration subtests `BuildPastItsTimeoutKeepsTheMap…` and `AGoListFailureKeepsTheMapsEntries…`. |
| No requeue at the failed HEAD (D5) | The "next pass" assertion in `BuildPastItsTimeout…`, plus `Map.FailedAt` in `TestMapFailedAt`. |
| Failure-only map for a package with no entries (D5) | The `broken` fixture, via `wantBrokenFailed` and the marker check. |
| Self-check on own-package block keys (D7) | `ANewTestIsMeasuredAloneAndAKeptEntryTheBuildCannotPlaceIsReMeasured`. |
| Self-check on linked functions, via `ProfiledFuncs` and the `lostFunc` branch | None. Only the block-key branch is pinned. |
| Audit runs after the queue is empty, re-measures the oldest entry, counts a differing result, only on a free slot (D7) | `TheAuditReMeasuresTheOldestEntryAndCountsADifferentResult` pins oldest-first and the differing count, with one entry in the test. |
| Audit skipped while work is queued, equal result not counted as a mismatch, slot not free | None. `runTestMapIn` sets `auditTests` to 0, so `CapLeavesTheRestQueued` never reaches it. |
| `TryAcquire` | `TestTryAcquire_TakesAFreeSlotAndNeverWaits`. |
| Held build lock skipped and counted as queued | `lockedSkip` with `Queued: 4` in `CapLeavesTheRestQueued`. |
| Plan step holds no gate slot | None. |
| `test_map_max_age_days` retired | The `TestLoadConfig` row "a retired test_map_max_age_days still loads". |
| Exit record fields and tolerance to an old `unbuilt` record (D12) | `TestWriteExitRecordReadExitRecord`. Integration reads the plan counts from the record and checks planned HEAD, queued and the mismatch counts. |
| Verb envelope prints `plan` and `packages` (per-package build seconds, run seconds, test count) | None. `testMapOutcome` decodes neither. Nothing asserts that `PackageMeasure` is filled. |
| Plan-failure fallback (mapped-test scan or `PlanUpkeep` error queues a whole job) | None. |
| Non-whole job whose map no longer stands at HEAD is skipped and counted as queued | None. |

**Findings**

1. MEDIUM, design. `internal/gatecli/testmap_integration_test.go` (`HeadMoveRetriesTheFailedBuild…`, around line 683): no test covers a non-body miss, or any miss on a map that holds entries. The whole rebuild must start from an empty map. The acceptance criterion "after the rebuild's first round the map at HEAD holds no entry measured before HEAD" has no row. If `mapPackage` seeded `m` from the old map, no test would fail. Evidence: `testmappackage.go` `mapPackage`, the `job.whole` branch.

2. MEDIUM, design. `internal/gatecli/redundancy_integration_test.go:170-176`: nothing exercises a map that holds entries plus a `Failure` field. `mappedPackages` was changed to judge such a map by its entries (D5), and only the failed-outcome row pins the cover rule. The spawn-marked-not-a-cover half of D4 has no row either.

3. MEDIUM, design. `internal/gatecli/testmappackage.go` (`runTest`, `runTests`, `fillMapTests`): the failed, skipped and no-coverage outcome entries have no assertion. Only the timed-out outcome is pinned. The D4 rule that a run ended by signal or stop records nothing is also unpinned.

4. MEDIUM, design. `internal/gatecli/testmappackage.go` (`selfCheck`, `lostFunc`) with `internal/testcover/blockkey.go` (`ProfiledFuncs`): the self-check's linked-function half has no test. A kept entry whose linked function is absent from the fresh build is not covered. D7 requires both halves.

5. LOW, design. `internal/gatecli/testmapaudit.go`: the audit's guards have no row. These are skipping while the queue is non-empty (`mapPackages` returns before `audit` when `r.queued > 0`), not running when `takeSlot(false)` finds no free slot, and not counting an equal result as a mismatch. Only the happy path is pinned.

6. LOW, design. `internal/gatecli/testmap.go` (`runTestMap` envelope): the new `plan` and `packages` keys (per-package build seconds, run seconds, test count) are not asserted. `testMapOutcome` decodes neither, and the exit record's `Packages` is only round-tripped in `TestWriteExitRecordReadExitRecord`, not filled by a real call.

7. LOW, design. `internal/gatecli/testmapplan.go:70-73,82-84` and `testmappackage.go` (the `planned.Commit != head` skip): the three fallback paths have no row. These are the mapped-test scan error and the `PlanUpkeep` error, each of which queues a whole rebuild, and the non-whole job whose map no longer stands at HEAD, which is skipped and counted as queued.

8. LOW, design. No test pins that the plan step holds no gate slot, a PATTERN-gate-slot constraint.

**Tool calls**
1. `git diff 46c3f64b..HEAD -- internal/gatecli/testmap_integration_test.go internal/gatecli/redundancy_integration_test.go`: the manifest had not read the integration tests.
