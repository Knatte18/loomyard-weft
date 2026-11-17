```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and new branch against the planned test rows. One criterion and several branches have no row, and two items are carried by no card.

**MEDIUM**

1. **D1: a detached `--map` call writes the map.** No row covers this criterion.
   - Card 1's `TestTestRequestWords_Map` checks only that `--map` appears in the words and that the two `detachKeyDir` keys differ.
   - No row runs a detached map call through the runner's replay of `test --wait` plus the words, so nothing shows the replayed run parses `--map` and writes the map.
   - Evidence: `_lym/plan/01-gate-test-map-mode.md`, the Tests bullet for `TestTestRequestWords_Map`; the decision record's D1 criterion "the detached map call writes the map".
2. **D6: a dirty package whose coverage build failed.** No row reaches this check.
   - Card 4 checks "once the coverage build ended, compiled or failed … before a build failure is written".
   - Every planned dirty-file row has a build that compiles: `ADirtyLinkedPackage…` and `ALinkedFileDirtied…`. `AFailedListingThroughADirtyFileWritesNoMap` covers the failed-listing path, not a failed build with a known linked set.
   - So no row shows that a failed build over a dirty tree leaves the map unwritten instead of recording the failure.
   - Evidence: `_lym/plan/04-map-run-clean-checks.md`, the third `mapPackage` check bullet.
3. **D6: the check over a package's own tree when its test or spawn scan fails before any build.** No row reaches it.
   - `ASpawnScanFailureWritesAFailedMapWithTheMarker` runs over a clean tree.
   - Evidence: `_lym/plan/04-map-run-clean-checks.md`, the second `mapPackage` check bullet.
4. **D1: `--card` scope and `--main-packages`.** No row reaches either.
   - Card 1's `packages(sc)` handles the share-union branch, and D1 says `--main-packages` "applies with `--card` as for any call".
   - Only `--base` (`ABaseScopeMapsThePackagesItsDiffTouches`) reaches the share-union branch.
   - Evidence: `_lym/plan/01-gate-test-map-mode.md`, the bullet "any other scope is the union over tier0's and tier1's shares".
5. **Card 1: three explicit-target branches.** No row reaches any of them:
   - the `./...` and `/...` expansion through `modulePackages`;
   - the filter keeping a share's targets inside the module root;
   - the computeScope error return in `runTestMap`.

   Evidence: `_lym/plan/01-gate-test-map-mode.md`, the explicit-targets and union bullets of `testMapRun.packages(sc)`.
6. **D7: no green record stands in for a measurement.** No row tests this.
   - No row runs a map call after a plain call left a green record, to show the map run still measures.
   - The plan relies on the map run calling no subtraction.
   - Evidence: the decision record's D7 sentence "A map run subtracts no green tier either"; `_lym/plan/03-map-run-tier-bounds.md`, the Slots bullet "calls no green subtraction, as card 1 left it".

**LOW**

7. **D1 and D4: the tier0 halves of two refusals.** Each refusal has a tier0 and a tier1 branch, and the plan reaches only one of each.
   - The go-json refusal is reached only for tier1 (the `NotGoJSON` row).
   - The memory-below-cap refusal is reached only for tier0 (the `MemoryBelowTheTestCap` row).
   - A loop bug that checks one tier only would pass.
   - Evidence: `_lym/plan/01-gate-test-map-mode.md:105-106`; `_lym/plan/03-map-run-tier-bounds.md:52-53`.
8. **D7: the "writes a log" sentence.** No card makes the map run write a log of its own.
   - Card 3 only sends `passed_alone` through `logger.Info`, and card 1's Bound speaks of "its log" without creating one.
   - D5 also asks that the list appear "in the call's output and log", and no row checks the log side.
   - Evidence: `_lym/plan/01-gate-test-map-mode.md:128`; `_lym/plan/03-map-run-tier-bounds.md:61`.
9. **Constraint `PATTERN-child-processes`.** No card's Intent carries it.
   - The constraint requires every test process and build to be logged at spawn and waited on, a cancelled one included.
   - The only cancellation coverage is the retained `ACallEndedDuringABuildOrATestWritesNothing` and `AnAheadBuild…` rows, which check waiting, not logging.
10. **Card 3: the exit-1 branches are not isolated.** Each branch is reached only beside another cause of exit 1.
    - The timed-out exit-1 path is reached only alongside failed tests in the first `--all` call (`TestAlphaHangs` and `TestOutcomesFails`).
    - The build-failure exit-1 path is reached only alongside failed tests in that same call (`broken`), and in Linked's GoList row.
    - `BuildPastItsTimeoutKeepsTheMapAndIsNotRetriedAtItsHead` has no stated exit, although card 3 says "each subtest asserts the exit it expects".
    - Evidence: `_lym/plan/03-map-run-tier-bounds.md:84,92-93`.
11. **D6 criterion "a package with a modified file".** Only untracked files are planned.
    - Every planned dirty row uses an untracked file (`notes.txt`, `scratch.txt`, the extra file in `emptied`).
    - No row covers a modified tracked file.
12. **D5: kill-then-rerun.** No row covers a test killed beside others, by the guard or its cap, that then passes alone.
    - Only failure beside others is reached (the Excludes tests).
    - The guard row (`TestGuardedHoldsMemory`) fails alone too.
13. **Card 1: `notests` in the first `--all` call.** The card does not say what to expect for it.
    - The old subtest expected `notests` built.
    - Under D1's package set it has neither a map nor a test, so it drops out.
    - The card names `dirty` and `ignored` as dropped but not `notests`, so the implementer must guess.
    - Evidence: `_lym/plan/01-gate-test-map-mode.md:136-138`.

Tool calls:
- I grepped `clihelp.GroupRunE` to confirm what the `TheRetiredVerb` row would see for `testmap`. My context lacked `internal/clihelp/exec.go`. It returns the error `unknown subcommand "testmap" for "gate"`, which fits the row.
