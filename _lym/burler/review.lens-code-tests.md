```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, mapping each added or changed behaviour to the test row that would fail if it broke.

**Behaviour → pinning test** (`itg` = `internal/gatecli/testmap_integration_test.go`)

| Behaviour | Pinned by |
|---|---|
| `--map` refused beside each partner | `cli_test.go` `TestGateTest_RefusesBeforeResolvingTheHub` rows |
| `--map` in the detached words and replay | `TestTestRequestWords_MainPackages` MapSet/MapUnset, `TheDetachedRunnersReplayWritesTheMap` |
| `testmap` is unknown | `Refusals/TheRetiredVerb` |
| outside hub, no mapped tier, counter, memory-below-cap, unknown package, card with no plan | `Refusals` rows |
| scope: targets, `--all`, `./...`, `--base`, `--card`, `--main-packages`, computed with no green base | the matching `TestGateTestMap` subtests, `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` |
| scope: `dir/...` suffix | none (F4) |
| package set: has a map, left-out reasons | linked-packages hand-written maps, `NamedPackageWithNoTrackedGoFileIsSkipped` |
| package set: unscannable tests still enter | none (F5) |
| replace map whole, last test deleted | `AMapRunReplacesAnExistingMapWhole`, `APackageWhoseLastTestWasDeletedGetsAMapWithNoTests` |
| build one ahead, pending build cancelled | `TestsRunSideBySide…`, `AnAheadBuildTheWalkNoLongerNeeds…` |
| side-by-side tests each get only their own blocks | none (F1) |
| spawn-marked and skipped entries | `TheMapRecordsLinkedFunctions…`, `AnAllCall…` |
| count bounded by the tier's `parallel`, per tier | none (F2) |
| count bounded by `memory_mb` / cap | `TestMapSideBySide`, `ATiersMemoryMBClamps…` |
| tiers never overlap | `ATiersTestsRunBeforeTheNextTiers` |
| tier0 runs before tier1 | none (F6) |
| guard kill named, run goes on | `ATestTheMemoryGuardKillsIsNamed…` (Linux) |
| per-test memory cap | `ATestPastTheMemoryCapIsRecordedAsFailed` |
| passed-alone rerun and list | `TestsRunSideBySide…` |
| no `lym` build | `AMapRunBuildsNoLym` |
| slot ask, enclosing slot | `TestMapSlotAsk`, `AMapRunHoldsTheLargerSlotAsk…`, `ACallInsideAnEnclosingGateRunsSlot…` |
| no run record, green not subtracted | `AMapRunWritesNoRunRecordAndAGreenRecord…` |
| exit 1 on failed test, timeout, build failure; map still written | `AnAllCall…`, `NamedPackageIsRebuiltWhenFresh`, `BuildPastItsTimeout…` |
| failed build keeps entries | `BuildPastItsTimeout…`, `AGoListFailure…` |
| prime vs worktree maps dir | `AnAllCall…`, task-worktree test |
| check before write (no tests, all spawn-marked, failed scan, failed build, failed listing) | the matching dirty-file subtests |
| check after build stops tests from running | none (F3) |
| failed listing, module below the worktree root | none (F7) |
| HEAD moved ends the call | zmoved, `ALinkedFileChanged…`, `AnAheadBuild…` |
| `redundant` ways forward | `TestGateRedundant` refusal rows |
| signal and cancel write nothing | `TestGateTestMapCLI`, `ACallEndedDuringABuildOrATest…` |

**Findings**

- **F1. MEDIUM, design.** `itg:1009` (`AnAllCallMapsEveryPackageWithAMappedTest`)
  - No test pins D3's acceptance criterion that two tests measured side by side each get only their own blocks.
  - The comment "TestAlphaOne and TestAlphaTwo each keep their own blocks" rests on `alpha`, but `TestAlphaOne` is tier0 and `TestAlphaTwo` tier1 (`itg:828`). The per-tier batches in `measureTests` never run them together.
  - The pairs forced to overlap (`meets`) both cover block 0, so workers sharing a profile path would go unnoticed. `outcomes` does not force an overlap.
  - Fix: force two same-tier tests with different coverage to overlap.
- **F2. MEDIUM, design.** `internal/gatecli/testmapstamp_test.go:71` (`TestMapSideBySide`)
  - Nothing pins D4's "never passes the running tier's `parallel`".
  - The `cappedTo` rows ignore the argument. The `asGiven` rows expect the memory quotient, which any parallel at or above it satisfies. So passing the wrong parallel to `capped` fails no row.
  - No integration subtest sets `parallel: 1`, or different counts for tier0 and tier1, and checks for no overlap. A swapped or call-wide `sideBySide` would pass.
- **F3. LOW, design.** `itg:1716` (`ADirtyLinkedPackageLeavesTheMap…`)
  - Card 4's "a package dirty at that point runs no test" is unpinned.
  - Deleting the `checkClean` between build and tests in `mapPackage` still passes, because `endJob` gives the same unmeasured reason.
  - Asserting that no coverage run was recorded (`testMapLymSeenEnv`) would pin it.
- **F4. LOW, design.** `itg:1136` (`AnEllipsisTargetMapsEveryPackageBelowIt`)
  - Only `./...` is tried. No row names a `dir/...` target.
  - `packagesBelow`'s subtree filter with `dir != "."` and the `CutSuffix` branch of `ellipsisRoot` could therefore break unseen.
- **F5. LOW, design.** `internal/gatecli/testmap.go` (`leftOutReason`)
  - No test has a package with no map whose test file does not parse. Such a package should enter the set and record the failure.
  - `garbled` breaks only its non-test source, so the spawn scan fails, not `mappedTests`.
  - Inverting `err == nil &amp;&amp; len(mapped) == 0` for the error case would silently leave the package out.
- **F6. LOW, design.** `itg:1358` (`ATiersTestsRunBeforeTheNextTiers`)
  - The name says tier0 runs before tier1, but the subtest only shows the two do not overlap. Reversing `mappedTiers` order in `measureTests` still passes.
- **F7. LOW, design.** `itg:1830` (`AFailedListingThroughADirtyFileWritesNoMap`)
  - It runs only with the module at the worktree root (`moduleRel == "."`).
  - `dirtyModuleFile`'s `moduleRel+"/"` filter (a dirty file under a subdirectory module blocks the write, one outside it does not) has no row.

**Tool calls**

1. Bash `sed` over `itg` lines 355–375, 405–425 and 615–1250: the manifest held only the file head and test index.
2. Bash `sed` over `itg` lines 1250–1942: the rest of the test bodies, not in context.
3. Bash over `testmapstamp_test.go` and the diff of `redundant_integration_test.go`: neither was in context.
4. Read of the persisted output of call 1: it was truncated.
5. Read of the persisted output of call 2: it was truncated.
