```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For every guard, refusal, branch and test row the cards add, I traced whether a real entry point can reach it.

**Per guard, reachable or not**

Card 1:
- **`--map` with each partner flag** (`--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--tags`, `-C`, words after `--`): reachable. `cappedCall` runs `flagShapeRefusal` first, the card puts the `--map` check before the `--tags` and `-C` checks, and `absoluteDir` only runs after it returns empty (run.go:43, detachfront.go:258-268).
- **Tier counter not `go-json`** (rows `NotGoJSONTier0`/`NotGoJSONTier1`): reachable. The config admits `counter: vstest` (gateslot/tiers.go:247-252).
- **A `computeScope` error as the run's refusal, row `ABadBase` (`--base nosuch`)**: NOT reachable through that input. See finding 2.
- **Narrowing to the union of tier0's and tier1's shares** (`ABaseScopeMapsThePackagesItsDiffTouches`, `ACardScopeMapsItsCardsPackages`, the task-worktree `--base HEAD~1` call): NOT reachable under the fixtures' gate config. See finding 1.
- **`left_out` reason `no Go file tracked at HEAD`**: reachable (the `ignored` package with its new test file).
- **`left_out` reason `no tier0 or tier1 test and no map`**: no named test reaches it. See finding 4.
- **Unknown-package and outside-hub refusals**: reachable (existing rows, now run as `test --wait --map`).
- **`TheRetiredVerb`**: reachable (clihelp/exec.go:235).
- **`TheDetachedRunnersReplayWritesTheMap`**: reachable (detachrun.go:373 replays `test --wait` plus the words).

Card 3:
- **`memory_mb` below `redundancy_test_memory_mb`** (rows `MemoryBelowTheTestCapTier0`/`Tier1`): reachable. 64 and 128 both pass config validation (tiers.go:241-245).
- **`ATiersMemoryMBClampsItsTestsToOneAtATime`** and **`ATiersTestsRunBeforeTheNextTiers`**: reachable.
- **`ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn`**: reachable. One tree passes `memory_quit_mb`.
- **`ATestTheGuardKillsBesideAnotherPassesAloneAndIsListed`**: NOT reachable. See finding 3.
- **Collides alone-rerun and `passed_alone`**: reachable, but fragile. See finding 5.
- **`AMapRunHoldsTheLargerSlotAskOfItsTiers`**: reachable. `AcquireSlots` with count 0 clamps to every slot (pool.go:190).
- **No-run-record subtest, exit 1 on `failed_tests`, `AMapRunBuildsNoLym`**: reachable.

Card 4:
- **HEAD-moved reason** (zmoved and amoves rows, the LandsLinked rows): reachable.
- **Dirty owned file after the build** (`linked/linked.go`, `scratch.txt`): reachable.
- **`trees == nil` falling back to `dirtyModuleFile`** (`AFailedListingThroughADirtyFileWritesNoMap`): reachable.
- **Failed build over a dirty linked file; spawn-scan failure over a dirty file; package with no mapped test; package whose tests are all spawn-marked**: reachable.
- **Check before a write after a failure while the tests run** (profile reduction): no named test. See finding 6.
- **Does the guarded write site recur unguarded?** `measureTests` calls `writeMap` directly (testmappackage.go:317). Card 4 routes that write through `write`, and `writeFailed`'s kept-map branch writes after its check. Held.

Card 5:
- **Rows `nomap`, `badindex`, `broken`, `stale`, `lacking`**: reachable through the existing fixture (redundant_integration_test.go:216-228, 357-363).

**Findings**

1. **MEDIUM**, design. Card 1, Tests (`ABaseScopeMapsThePackagesItsDiffTouches`, `ACardScopeMapsItsCardsPackages`, the task-worktree `--base HEAD~1` call).
   - **Summary:** under these fixtures the narrowing branch that unions tier0's and tier1's shares is never reached. Both subtests would map every package, so they fail.
   - **Evidence:**
     - The template's default is `scoper: none` (internal/gateslot/template.yaml:40).
     - The fixtures' gate config sets no `scoper` (testmap_integration_test.go:725-728, 668-669).
     - With a scoper other than `go`, `computeScope` returns `scope{form, whole: true}` for the `--base`, computed and `--card` forms alike (scope.go:186-188).
     - The task-worktree call passes anyway, because `solo` is that module's only package, so it pins nothing.
   - **Fix:** the card's fixtures set `scoper: go`.

2. **MEDIUM**, design. Card 1, Tests, `Refusals` row `ABadBase` (`--base nosuch`, wanting exit 1).
   - **Summary:** a bad `--base` is never a `computeScope` error. It falls back to a whole scope, so the row cannot reach the refusal it names.
   - **Evidence:**
     - Under `scoper: none` the form is simply whole (scope.go:186-188).
     - Under `go`, `diffBaseGroups` returns the base as given (scope.go:358-359).
     - `DeriveTargets` then turns a non-ancestor base into `wholeTargets(...)` with a nil error (impactset/targets.go:63-65).
   - **Fix:** a reachable `computeScope` error is an unknown `--card` id or a worktree with no plan (scope.go:128-130, 165-178).

3. **BLOCKING**, design. Card 3, Tests, `ATestTheGuardKillsBesideAnotherPassesAloneAndIsListed`.
   - **Summary:** the state this row needs cannot occur. With `memory_quit_mb` above one hoard and below two, the guard kills neither test, so neither lands in `passed_alone`.
   - **Evidence:**
     - The guard's limit applies to each test binary's own tree (memoryguard.go:38-42, 96-99).
     - Its pressure trigger kills only a tree holding at least `pressureKillFloor`, 1 GiB, and only under a cgroup limit (memoryguard.go:27-33).
     - One hoard is `testMapHoardMegabytes` = 384 MB (testmap_integration_test.go:602).
   - **Consequence:** no other planned row reaches the "killed by the guard beside others, passes alone" path in D5.

4. **LOW**, design. Card 1, Intent, `testMapRun.packages`, `left_out` reason `no tier0 or tier1 test and no map`.
   - **Summary:** no named test reaches this branch.
   - **Evidence:** card 1 drops the `notests` package from the fixture (01-gate-test-map-mode.md:142). Only `NamedPackageWithNoTrackedGoFileIsSkipped` checks `left_out`, and only for the other reason (line 148).

5. **LOW**, design. Card 3, Tests, the `Collides` stand-in (`testMapCollideWait` = 2 s).
   - **Summary:** an alone run of a Collides test passes only after waiting 2 s, under a per-test bound of 3 s. Under load it can time out instead of passing, so the `passed_alone` assertions can fail without any defect in the code.
   - **Evidence:** the fixture's test bound is 20 × 150 ms (testmap_integration_test.go:589, 721).
   - **Also unspecified:** whether a Collides test removes its record after its run. If records stay, a later alone rerun sees the sibling's record and fails (the existing `runsApart` removes its own record, testmap_integration_test.go:411).

6. **LOW**, design. Card 4, Intent, the write-path bullet "a failure while the tests run, such as a profile that cannot be reduced, is checked over the linked trees".
   - **Summary:** this guarded write path has no named input or test that reaches it.
   - **Evidence:** `measureTests` reaches `writeFailed` on a `runBatch` error (testmappackage.go:278-283). Every new card-4 subtest reaches a different write path (04-map-run-clean-checks.md:61-69).

**Tool calls**
- Read the memory guard's kill rule in memoryguard.go (grep), to tell whether `memory_quit_mb` applies per binary or to the whole tree. The manifest read only `guardOwnTests` and `startMemoryGuard`.
- Read the gateslot config's `scoper` and `counter` validation and `diffBaseGroups` (grep), to see how the fixtures' scope forms and a bad `--base` resolve. The manifest lacked these.
- Read the template's default `scoper`, `impactset.DeriveTargets`'s handling of a non-ancestor base, and `allInertSince` (grep), to trace the `ABadBase` input. The manifest lacked these.
