```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**
- D1 flag refusals (`--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--tags`, `-C`, words after `--`): the eight new rows in `TestGateTest_RefusesBeforeResolvingTheHub`, `cli_test.go:39-46`. Each wants `lym gate test --map` in the error.
- D1 non-`go-json` tier 0 or tier 1: `TestGateTestMap/Refusals` rows `NotGoJSONTier0` and `NotGoJSONTier1`, `testmap_integration_test.go:966-974`.
- D1 no mapped tier, outside a hub, unknown package, card with no plan: `Refusals` rows at lines 954-1002.
- D1 `testmap` verb gone (D9): `Refusals/TheRetiredVerb`, line 1006.
- D1/D2 package set and scope forms: subtests `AnAllCallMapsEveryPackage…`, `ABaseScope…`, `AnEllipsisTarget…`, `ACardScope…`, `NamedPackageWithNoTrackedGoFileIsSkipped` (the `left_out` case).
- D1 `--map` in the detached call's words: `TestTestRequestWords_MainPackages` rows `MapSet` and `MapUnset`, `detachfront_test.go:461`.
- D1 detached replay writes the map: `TestGateTestMapCLI/TheDetachedRunnersReplayWritesTheMap`, line 2000.
- D2 whole replace, and a package whose last test was deleted: `AMapRunReplacesAnExistingMapWhole`, `APackageWhoseLastTestWasDeletedGetsAMapWithNoTests`.
- D2 pending build cancelled and waited on: `AnAheadBuildTheWalkNoLongerNeedsIsCancelledRatherThanWaitedFor`. Build bound: `BuildPastItsTimeoutKeepsTheMapAndIsNotRetriedAtItsHead`.
- D3 spawn-marked and skipped entries, no `lym` built: `TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker`, `AMapRunBuildsNoLym`.
- D4 count formula `memory_mb ÷ cap`: `TestMapSideBySide` rows, `testmapstamp_test.go:71`. End to end: `ATiersMemoryMBClampsItsTestsToOneAtATime`. A broken clamp would put the colliding tests in `passed_alone`.
- D4 `memory_mb` below the cap refused with both values: `Refusals` rows `MemoryBelowTheTestCapTier0` and `MemoryBelowTheTestCapTier1`, lines 982-990. The wording includes `lym config gate`.
- D4 tier 0 before tier 1: `ATiersTestsRunBeforeTheNextTiers`.
- D4 guard kill named, run goes on: `ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn`. Cap kill recorded as failed: `ATestPastTheMemoryCapIsRecordedAsFailed`.
- D5 alone-rerun and `passed_alone` list: `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone`.
- D6 modified or untracked file, HEAD moved, failed listing, failed build over a dirty linked file, spawn-only and no-test packages, spawn-scan failure over a dirty file: the `TestGateTestMapLinkedPackages` subtests at lines 1759-1955, plus `APackageWithNoMappedTestIsCheckedBeforeItsWrite`. HEAD-moved exit 1 with the unmeasured list is in the `AnAllCall…` subtest, lines 1040-1044.
- D7 slot ask: `TestMapSlotAsk` for the arithmetic, and `AMapRunHoldsTheLargerSlotAskOfItsTiers` end to end.
- D7 no records, and a green record does not stand in for a measurement: `AMapRunWritesNoRunRecordAndAGreenRecordDoesNotStandInForIt`.
- D7 failed test still writes the map with exit 1: the `AnAllCall…` subtest and `redundant_integration_test.go:239` (exit 1 after the map is written).
- D7 failed build keeps old entries: `AGoListFailureKeepsTheMapsEntriesAndRecordsTheFailureWithHead`, `AnAllCall…`.
- D8 task-worktree maps: `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`.
- D9 redundant refusal way-forward text: the new `wantWayForward` field in `TestGateRedundant`, `redundant_integration_test.go:354-395`. It covers no map, failed-build-only map, unreadable map, and both stale rows including "commit or revert".
- Refactors with no behaviour change: `blockkey.go` `funcsHolding` is covered by the existing `CoveredFuncs` tests. The deletions removed only code with no live reader.

**Findings**
1. **LOW, design.** `mapSideBySide` is documented as using the tier's `parallel` with the per-machine `tier_parallel` override applied. The map-run tests never set a `tier_parallel` entry, so an override that stopped reaching `r.tiers[n].Parallel` would break no map-run test. Evidence: `testmap.go:233-243` and `testmapstamp_test.go:71-106`; `tier_parallel` appears in tests only at `test_integration_test.go:2084` (config validation, outside the map run). A row in `TestMapSideBySide` is not the fix, since it takes a `Tier`. It needs a `tier_parallel` row in a map-run integration test, or a `prepare`-level assertion.
2. **LOW, design.** D1 says a map call and a plain call over the same targets never share a detached run. The only test is the words row (`detachfront_test.go:461`), which shows `--map` is in the replay words. Nothing shows the detached key (join or replay identity) differs between a map call and a plain call over the same targets. If the key is derived from `words()` this is covered indirectly. I did not read the key function, so I cannot confirm that.

No other behaviour the diff adds lacks a test row.

**Tool calls (3 used)**
1. `grep`/`git diff` of subtest names and the diffs of `cli_test.go`, `detachfront_test.go` and `redundant_integration_test.go`: the manifest did not hold the integration test names.
2. `grep` of `testmap_integration_test.go` for HEAD-moved, `passed_alone`, `unmeasured`, `memory_mb` and `tier_parallel` assertions: to confirm D5, D6 and D4 coverage.
3. `sed` of lines 1307-1345 and 1454-1482, plus a `tier_parallel` grep across `gatecli` tests: to confirm what the D5 and D4 rows assert, and whether `tier_parallel` is exercised.
