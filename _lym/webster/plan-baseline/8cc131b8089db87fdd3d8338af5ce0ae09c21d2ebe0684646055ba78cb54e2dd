# Card 3 — map-run-tier-bounds

**Edit:**
- `internal/gatecli#testMapRequest`
- `internal/gatecli#testMapRun`
- `internal/gatecli#runTestMap`
- `internal/gatecli#testMapRun.prepare`
- `internal/gatecli#testMapRun.mapPackage`
- `internal/gatecli#testMapRun.measureTests`
- `internal/gatecli#testMapRun.runBatch` -> `func (r *testMapRun) runBatch(p *packageRun, queue []testcover.TestFunc, sideBySide int) ([]mappedRun, error)`
- `internal/gatecli#testMapRun.runSideBySide` -> `func (r *testMapRun) runSideBySide(p *packageRun, tests []testcover.TestFunc, sideBySide int) ([]mappedRun, error)`
- `internal/gatecli#testMapRun.holdSlot`
- `internal/gatecli#testMapRun.takeSlot`
- `internal/gatecli#testMapRun.runTest`
- `internal/gatecli/prebuild.go`
- `internal/gatecli/doc.go`
- `internal/gatecli#TestGateTestMap`
- `internal/gatecli#TestGateTestMapLinkedPackages`
- `internal/gatecli#TestGateRedundant`
- `internal/gatecli#runTestMapStandIn`
- `internal/gatecli#runTestMapStandInLymBuild`
- `internal/gatecli#runTestMapIn`
- `internal/gatecli#decodeTestMap`
- `internal/gatecli#callTestMapCounts`
- `internal/gatecli#testMapOutcome`
- `docs/overview.md`
- `internal/gateslot/doc.go`
- `pattern/PATTERN-gate-slot.md`
- `pattern/PATTERN-test-speed.md`

**Delete:**
- `internal/gatecli#testMapRun.buildLym`
- `internal/gatecli#testMapHangLymBuildEnv`

**Create:**
- `internal/gatecli#mapSideBySide`
- `internal/gatecli#mapSlotAsk`
- `internal/gatecli#TestMapSideBySide`
- `internal/gatecli#TestMapSlotAsk`
- `internal/gatecli#testMapCollideEnv`
- `internal/gatecli#collidesWithAnother`

**Uses:**
- `internal/gatecli#testMapMeetWait`
- `internal/gatecli#memoryCappedParallel`
- `internal/gatecli#guardOwnTests`
- `internal/gatecli#startMemoryGuard`
- `internal/gatecli#runTest`
- `internal/gateslot#Tier.MemoryLimit`
- `internal/gateslot#Pool.AcquireSlots`
- `internal/gateslot#Config.RedundancyTestMemory`
- `internal/gateslot#ReadRunRecords`
- `internal/gateslot#RunLogDir`

**Intent:** Bound the map run's test processes by each tier's parallelism and memory, rerun a test that failed beside others alone and report it, hold the gate slots the mapped tiers ask for, build no `lym`, and exit non-zero on a failed measurement (D3, D4, D5, D7).

Parallelism and memory (D4):
- `mapSideBySide(tier, testMemory, capped)` returns how many of the tier's tests run side by side: the tier's `Parallel` as the config gives it, this machine's `tier_parallel` entry already applied, clamped by `capped(parallel, testMemory)` as `lym gate redundant` clamps it to the headroom lym's cgroup leaves (one at a time where the memory cap does not cover a process tree).
  When the tier sets `memory_mb` it is also at most `tier.MemoryLimit() / testMemory`, whether or not lym's cgroup is limited.
- `testMapRun` replaces `parallel` and the one `sideBySide` count with `sideBySide map[int]int`, filled once per mapped tier in `testMapRun.prepare` from `mapSideBySide` with `request.cappedParallel` and `cfg.RedundancyTestMemory()`.
- `testMapRun.prepare` refuses a hub whose tier0 or tier1 sets `memory_mb` below `redundancy_test_memory_mb`, since that quotient is 0: `gate test --map: tier<n> sets memory_mb <m>, below redundancy_test_memory_mb <r>, so no test process fits its memory; way forward: raise tier<n>'s memory_mb or lower redundancy_test_memory_mb with lym config gate from the prime, then re-run the same command`.
  It checks tier0, then tier1.
  The input reaching it is a gate config with that tier's `memory_mb: 64` and `redundancy_test_memory_mb: 128`, which the config admits; the `MemoryBelowTheTestCapTier0` and `MemoryBelowTheTestCapTier1` rows of `TestGateTestMap`'s `Refusals` reach it.
- `testMapRun.measureTests` runs a package's tier0 tests first, then its tier1 tests, each through `runBatch` with that tier's count, so at most one tier's count runs at once.
  `runBatch` and `runSideBySide` take the count instead of reading one call-wide count.
- Each test process keeps the `redundancy_test_memory_mb` kernel cap, and the memory guard keeps sampling the run's own process tree, so a `.test` process past `memory_quit_mb` is killed, named on stderr and recorded as failed while the package's other tests go on.
  `testMapRequest` gains `guardFor func(ctx context.Context, limit int64, out io.Writer) (stop func(), guardKilled func(pid int) bool)`, the guard's starter; `runTest`'s request leaves it nil, which means `guardOwnTests`.
  A test passes a starter that calls `startMemoryGuard` with `applies` set, since a test runs under an enclosing `lym gate test` whose ancestry turns `guardOwnTests` off.
- The guard's pressure trigger, which can kill a test for its siblings' memory, needs a 1 GiB tree under a cgroup limit, so no test drives it.

Alone rerun and its report (D5):
- `runBatch` reruns alone each test that failed, timed out or was killed by the guard or its cap beside others; the alone run's outcome stands, as today.
  It also records each test whose alone run passed as `<package>.<Test>` in a new `testMapRun` field `passedAlone`.
- `runTestMap` prints `passed_alone` in its envelope and logs the same list through `internal/logger`, the run's log; a test that passed only alone enters the map as a plain pass.

No `lym` build (D3):
- `testMapRun.buildLym` goes with `testMapRun`'s `lymBuilt`, `lymBin` and `lymErr` fields and its call in `testMapRun.mapPackage`, and `testMapRun.runTest` sets no `gateslot.PrebuiltLymEnv`.
  `gateEnv` already strips an inherited one, so no measured test reads the variable; a tier0 test spawns nothing, and `testcover.ScanSpawns` marks every other test that reaches `internal/testkit/lymbin` as spawn-marked, which is never run.
- `prebuild.go`'s file comment names only the tagged `lym gate test` run.

Slots (D7):
- `mapSlotAsk(tiers)` returns the larger of tier0's and tier1's `Slots`, where 0, a tier asking for every slot, ranks above any count and is returned as 0.
- `testMapRun.holdSlot` and `testMapRun.takeSlot` acquire that many slots through `gateslot.Pool.AcquireSlots` for the whole call, inside an enclosing gate run's slot taking none as today; the builds keep `-trimpath` and `-p` from tier0's parallel through `gateslot.HolderEnv`.
- The map run writes no run record, green record, verify record or failure record, and calls no green subtraction, as card 1 left it.

Exit (D7):
- `runTestMap` prints its envelope and returns 1 when any package's build failed or any entry it wrote records a failed or timed-out outcome after the alone rerun; it lists those tests as `<package>.<Test>` in a new `failed_tests` key.
  It still writes every package's map, failed entries included, and a failed build keeps the map's old entries and records the failure, as today.

Tests:
- `TestMapSideBySide` and `TestMapSlotAsk` go in `internal/gatecli/testmapstamp_test.go`, whose tests are tier0.
  That file's header then names the map run's side-by-side count and slot ask beside its environment stamp;
  no target carries the header, since `edit-test-file` refuses a `_test.go` file's self glyph, so this sentence alone carries the change.
- `TestMapSideBySide`, tier0, a table over `mapSideBySide` with a stub `capped`: no `memory_mb` gives the capped parallel; `memory_mb` of twice the test cap gives at most 2; `memory_mb` equal to the test cap gives 1; a capped count below the memory quotient wins.
- `TestMapSlotAsk`, tier0, a table: tier0 alone; tier0 1 and tier1 2 gives 2; tier1 asking every slot gives 0; tier0 asking every slot beside tier1 3 gives 0.
- `testMapOutcome` gains `PassedAlone`, `FailedTests`, `Exit` and `Stderr` (the last two not decoded: `callTestMapCounts` sets them); `runTestMapIn` and `TestGateTestMap`'s `request` closure give the request a stderr buffer, which `callTestMapCounts` copies into `Stderr`.
  `decodeTestMap` fails only on a refusal envelope, so a run that exits 1 with its report decodes, and each subtest asserts the exit it expects.
- `runTestMapStandIn` gains two test suffixes:
  `HoldsMemory`, which keeps `testMapHoardMegabytes` and then sleeps past the guard's one-second sampling before it passes;
  `Collides`, which through `collidesWithAnother` keeps its record in the directory `testMapCollideEnv` names after it ends.
  A test whose record is already there is the alone rerun and passes at once;
  otherwise it records itself and waits for another test's record, bounded by `testMapMeetWait` as `meetsAnother` is, and fails as soon as one appears.
  Two Collides tests started side by side both fail, and their alone reruns both pass, with no time window deciding either outcome; the alone pass waits for nothing, so it sits far below the fixture's per-test bound of 20 times 150 ms.
  A Collides test run alone with no earlier run would wait out `testMapMeetWait`, so only the D5 subtest, which runs them side by side first, uses the suffix.
- `runTestMapStandInLymBuild` drops its hang branch with `testMapHangLymBuildEnv`; it keeps recording each build, which `AMapRunBuildsNoLym` reads.
- `TestGateTestMap`:
  - `Refusals` gains `MemoryBelowTheTestCapTier0` and `MemoryBelowTheTestCapTier1`.
  - The first `--all` call's subtest wants exit 1, with `outcomes.TestOutcomesFails` and `alpha.TestAlphaHangs` in `failed_tests`, outcomes' map written with the failed entry, and broken's failed build (D7).
  - `NamedPackageIsRebuiltWhenFresh` wants exit 1 with `alpha.TestAlphaHangs` alone in `failed_tests`, a timeout alone causing the exit.
  - `BuildPastItsTimeoutKeepsTheMapAndIsNotRetriedAtItsHead` wants exit 1 with `failed_tests` empty, a failed build alone causing the exit.
  - `AMapRunReplacesAnExistingMapWhole` wants exit 1, since outcomes' `TestOutcomesFails` fails.
  - `AnEllipsisTargetMapsEveryPackageBelowIt` wants exit 1, since `./...` holds outcomes, alpha's `TestAlphaHangs` and broken's failed build.
  - `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` replaces the excludes package's tests with `TestExcludesFirstCollides` and `TestExcludesSecondCollides`, and wants both in `passed_alone` (D5).
  - New subtest `ATiersMemoryMBClampsItsTestsToOneAtATime`: tier0 sets `memory_mb` equal to `redundancy_test_memory_mb` with `parallel: 2`, and a committed package `clamped` with `TestClampedFirstExcludes` and `TestClampedSecondExcludes` both pass with `passed_alone` empty (D4).
    An Excludes test run one at a time always passes, and two overlapping within its window fail beside each other and land in `passed_alone`.
  - New subtest `ATiersTestsRunBeforeTheNextTiers`: a package with one tier0 Excludes test and one tier1 Excludes test at `parallel: 2` both pass with `passed_alone` empty (D4).
  - New subtest `ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn`: `memory_quit_mb` below the hoarded memory, the guard started through `guardFor`, a package with `TestGuardedHoldsMemory` and `TestGuardedOne`; `Stderr` names the killed `.test` process, `TestGuardedHoldsMemory`'s entry is failed, `TestGuardedOne` covers block 0, and the exit is 1 (D4).
  - `PrebuiltLymIsBuiltOncePerCall` becomes `AMapRunBuildsNoLym`: under a `prebuilt_lym` tier no `lym` build is recorded and every run saw an empty `LYM_PREBUILT_LYM` (D3).
  - `ACallEndedDuringABuildOrATestWritesNothing` drops its `WhileTheLymBuildHangs` row and its `prebuilt_lym` gate setup.
  - New subtest `AMapRunHoldsTheLargerSlotAskOfItsTiers`: with the test holding one of the hub's two slots, a map call whose tier1 asks `slots: all` ends cancelled under a two-second context with no map written, and one whose tiers ask one slot each builds steady.
  - New subtest `AMapRunWritesNoRunRecordAndAGreenRecordDoesNotStandInForIt`, over the clean tree card 1 left, with tier0's command set to `true` and no green record yet: a map call over steady leaves `gateslot.ReadRunRecords` over the pool's `gateslot.RunLogDir` holding as many records as before it, and a following `runTest` with `--wait --tier 0 steady` runs its tier, its record not carried; that call records green, and a second map call over steady still measures it, its entry's `MeasuredAt` after the green call (D7).
  - `ATestPastTheMemoryCapIsRecordedAsFailed` wants exit 1.
- `TestGateTestMapLinkedPackages`'s `AGoListFailureKeepsTheMapsEntriesAndRecordsTheFailureWithHead` and `ASpawnScanFailureWritesAFailedMapWithTheMarker` want exit 1, and its other subtests exit 0.
- `TestGateRedundant` wants exit 1 from `test --wait --map cov`, since cov's `TestFails` fails, and reads the map it wrote.

Docs: `internal/gatecli/doc.go`'s `# Coverage maps` section states the per-tier side-by-side count and its memory clamp, tier0 before tier1, the alone rerun and `passed_alone`, the slot ask, no `lym` build and the exit code.
`docs/overview.md` and `internal/gateslot/doc.go`'s map-build bounds lines say the same.
`pattern/PATTERN-gate-slot.md`'s slot rule names the map run of `lym gate test --map` as a third place a gate build or test runs, holding the larger of tier0's and tier1's slot asks, where a tier asking for every slot ranks above any count.
`pattern/PATTERN-test-speed.md`'s coverage-map section states D4, D5 and D7.

**ImpactSummary:** Confined to the map run in gatecli: its slot count, per-tier parallelism, guard seam, alone-rerun report and exit code change, its `lym` build goes; `lym gate redundant`'s shared clamp and the tier runs are untouched, and `TestGateRedundant` reads the new exit.
