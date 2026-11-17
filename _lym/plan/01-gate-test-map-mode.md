# Card 1 — gate-test-map-mode

**Edit:**
- `internal/gatecli#newCommand`
- `internal/gatecli#testRequest`
- `internal/gatecli#runTest`
- `internal/gatecli#flagShapeRefusal`
- `internal/gatecli#testRequest.words`
- `internal/gatecli#testMapSite`
- `internal/gatecli#testMapRequest`
- `internal/gatecli#testMapJob`
- `internal/gatecli#testMapRun`
- `internal/gatecli#runTestMap`
- `internal/gatecli#testMapLocation`
- `internal/gatecli#coverageMapsDir`
- `internal/gatecli#testMapRun.prepare`
- `internal/gatecli#testMapRun.packages` -> `func (r *testMapRun) packages(sc scope) ([]string, string)`
- `internal/gatecli#testMapRun.mapPackages` -> `func (r *testMapRun) mapPackages(packages []string) error`
- `internal/gatecli#testMapRun.mapPackage` -> `func (r *testMapRun) mapPackage(job testMapJob) error`
- `internal/gatecli#testMapRun.measureTests`
- `internal/gatecli#testMapRun.addMeasure`
- `internal/gatecli#packageRun`
- `internal/gatecli#blockTable`
- `internal/gatecli#packageRun.newBlockTable`
- `internal/gatecli#stampEnvironment`
- `internal/gatecli/doc.go`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestTestRequestWords_MainPackages`
- `internal/gatecli#TestGateTestMap`
- `internal/gatecli#TestGateTestMapLinkedPackages`
- `internal/gatecli#TestGateTestMapCLI`
- `internal/gatecli#TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`
- `internal/gatecli#TestGateRedundant`
- `internal/gatecli#helperRoleTestMapCLI`
- `internal/gatecli#runTestMapCLIHelper`
- `internal/gatecli#startTestMapCLI`
- `internal/gatecli#runTestMapIn` -> `func runTestMapIn(prime, goBinary string, env []string, minuteUnit time.Duration, targets ...string) (int, []byte)`
- `internal/gatecli#callTestMap` -> `func callTestMap(t *testing.T, prime, goBinary string, env []string, minuteUnit time.Duration, targets ...string) testMapOutcome`
- `internal/gatecli#callTestMapCounts` -> `func callTestMapCounts(t *testing.T, prime, goBinary string, env []string, minuteUnit time.Duration, targets ...string) (testMapOutcome, testMapVerbCounts)`
- `internal/gatecli#testMapOutcome`
- `internal/gatecli#testMapVerbCounts`
- `cmd/lym#unboundedWaitFiles`
- `cmd/lym#testOnlyCodeAllowed`
- `README.md`
- `docs/overview.md`
- `internal/gateslot/doc.go`
- `internal/gateslot/template.yaml`
- `internal/dobbyengine/doc.go`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-daemon-wakeups.md`
- `internal/testcover#FuncIndex.funcsHolding` -> `func (x *FuncIndex) funcsHolding(blocks []ProfileBlock) ([]FuncKey, error)`
- `internal/testcover#FuncIndex.CoveredFuncs`

**Delete:**
- `internal/gatecli#testMapCmd`
- `internal/gatecli#testMapAuditTests`
- `internal/gatecli#testMapBuildLockSuffix`
- `internal/gatecli#testMapRun.measureJob`
- `internal/gatecli#testMapRun.orderJobs`
- `internal/gatecli#releaseBuildLock`
- `internal/gatecli#testMapRun.dirtyGoFile`
- `internal/gatecli#testMapBuildLockPath`
- `internal/gatecli#testMapRun.lockPackage`
- `internal/gatecli#testMapRun.planPackage`
- `internal/gatecli#testMapRun.planLockedPackage`
- `internal/gatecli#testMapRun.headSelector`
- `internal/gatecli#testMapRun.headTree`
- `internal/gatecli#keptEntryTest`
- `internal/gatecli#selfCheck`
- `internal/gatecli#entryCoverage`
- `internal/gatecli/testmapaudit.go`
- `internal/gatecli/testmapwait.go`
- `internal/gatecli/testmaporder_test.go`
- `internal/gateslot#Pool.TryAcquire`
- `internal/gateslot#TestTryAcquire_TakesAFreeSlotAndNeverWaits`
- `internal/testcover#FuncIndex.ProfiledFuncs`

**Uses:**
- `internal/gatecli#computeScope`
- `internal/gatecli#scope.forTier`
- `internal/gatecli#resolveRunDir`
- `internal/gatecli#counterGoJSON`
- `internal/gatecli#modulePackages`
- `internal/gatecli#holdsGoFile`
- `internal/gatecli#memoryCappedParallel`
- `internal/testkit/plankit#Write`
- `internal/planparser#PlanDir`
- `internal/testcover#WriteMap`

**Intent:** Make building the coverage maps a mode of `lym gate test`, set by `--map`, and retire `lym gate testmap` with its incremental planning, audit round, self-check of kept entries, per-package build lock and its wait, `--max` and run-record ordering (D1, D2, D8, D9's "unknown command").
The measure loop of `testmappackage.go` stays and becomes the map run; this card only reroutes it and drops the planning around it.

The flag and its routing:
- `newCommand` declares `--map` as a bool flag (`mapFlag`, a named constant `"map"` beside the flag code), stores it in a new `testRequest` field `mapMode`, and no longer registers `testMapCmd`, so `lym gate testmap` answers `unknown subcommand "testmap" for "gate"` through `clihelp.GroupRunE`.
  `testCmd`'s `Long` text gains one paragraph on `--map` and its examples gain `lym gate test --wait --map internal/foo`.
- `testRequest.words` emits `--map` after `--guards` and before the targets, so a detached map call's key (`detachKeyDir` hashes the words) differs from a plain call's over the same targets, and the runner's replay (`test --wait` plus the words, in `detachrun.go`) carries the flag; a map call never joins a plain call's run, nor the reverse.
- `flagShapeRefusal` checks a `--map` call first, before the `--tags` refusal, and refuses `--map` beside `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--tags`, `-C`/`--dir` or words after `--`.
  Each refusal names the flag and its way forward: `gate test: --map measures tier0 and tier1 over its scope, so it goes with none of --tier, --upto, --gate, --guards, --stop-on-fail, --tags, -C and words after --; way forward: drop <flag> and run lym gate test --map with targets or a scope flag`.
  `--tags` joins the list so a `--map --tags` call is sent straight to plain `--map`, not through the `--tags` refusal's `--tier` advice.
  The input reaching each is `lym gate test --map <flag> ./pkg`; the rows below reach them.
- `runTest`, right after `flagShapeRefusal`, returns `runTestMap(ctx, out, testMapRequest{...})` for a `mapMode` request: the request itself as the scope, `goBinary`, `os.Environ()`, `secondUnit`, `timeoutUnit` as the minute, `pipeWaitDelay` as the grace, `memoryCappedParallel` and `stderr`.
  Map mode never reaches the tier plan, the green subtraction, the shared build, the lints, the report steps or the guard step; the detached runner's card fixer needs `--gate` (`handsOff`), which `--map` refuses.

`testMapRequest` drops `maxPackages`, `packages`, `auditTests`, `waitBase` and `waitCeiling`, and gains `scope testRequest`, the call whose scope flags pick the packages.
`testMapJob` drops `whole`, `tests` and `audit`: every job is a whole rebuild.
`testMapRun` drops `selector`, `dirty`, `plan`, `selfCheckMismatches`, `auditMismatches` and `inert`, whose one reader was `headSelector`, and gains `cfg gateslot.Config`, the loaded gate config, and `leftOut []testMapSkip`.
`testMapSite` becomes `"lym gate test --map"`.

`runTestMap` drops the `--max` refusal and the planned-head, plan and mismatch keys of its envelope; it prints `built`, `skipped`, `failed`, `timed_out`, `left_out`, `head`, `queued` and `packages`.
`testMapLocation`'s and every other refusal of the run opens `gate test --map:`; the outside-hub refusal's way forward is `run lym gate test --map from a worktree of a hub`.
`coverageMapsDir`'s comment names `lym gate test --map`; the maps stay where they are, the prime's in the hub's shared maps and any other worktree's under its anchor `.lym` (D8).

`testMapRun.prepare` keeps loading the gate and env config and the module, drops the pre-walk status read and the `r.inert = cfg.Inert` assignment, and refuses a hub whose tier0 or tier1 counter is not `counterGoJSON`, naming the tier: `gate test --map: tier<n> counts its tests with <counter>, and a coverage map exists only for a go-json tier; way forward: when tier<n> runs Go tests, set its counter to go-json with lym config gate from the prime and re-run the same command; a hub whose tier<n> runs no Go tests has no coverage map to build`.
It checks tier0, then tier1, and refuses on the first that fails.
The input reaching it is a hub whose tier0 or tier1 sets `counter: vstest`, which the gate config admits; the `NotGoJSONTier0` and `NotGoJSONTier1` rows reach it.
A hub without tier1 maps tier0 alone, as `mappedTierConfigs` gives.

`runTestMap` computes the scope through `computeScope` with the loaded config, the location, the worktree root, the request's `scope` and a nil green record, so the scope is chosen as for any call, `--main-packages` with `--card` included, and the map run reads no green record at all (D7).
A computed scope is then the diff from the merge base with the parent branch, and the whole module when the worktree has no parent branch, whatever the green record holds.
A `computeScope` error is the run's refusal, as a plain call's is.
`testMapRun.packages(sc)` turns the scope into the package set, module-relative and sorted in import path order:
explicit targets resolve against the module root (`./...` and a `/...` suffix expand through `modulePackages`, and a target naming no package directory of the module is the existing refusal, reworded to open `gate test --map:`);
a scope whose `whole` is set, from `--all`, from a scoper other than `go` or from a computed or `--base` fallback, is every package `modulePackages` finds;
so is a scope where tier0's or tier1's share from `scope.forTier` has `whole` set;
otherwise the set is the union over tier0's and tier1's shares of each module's targets joined to the module's dir, keeping those inside the module root.
It reads the scope's shares directly, never `scopeInvocations`, since map mode ignores the tiers' commands.
`diffBaseGroups` derives the shares from `keptTiers` of the request, and a map request names no tier, so a computed or `--base` scope holds tier0's share alone.
Tier1's `forTier` then returns the call-wide scope, which for a narrowed scope has no modules and contributes nothing, and for a `--card` scope repeats tier0's modules.
That is the intended set: the packages the diff or the cards reach, picked once from the merge base or the `--base` commit.
A package enters the set only when it has a map (`testcover.MapPath` exists), or when it tracks a Go file at HEAD (`fileHashes` is not empty) and holds a tier0 or tier1 test (`mappedTests`).
Every other package is left out; one an explicit target names is listed under `left_out` with its reason, `no Go file tracked at HEAD` or `no tier0 or tier1 test and no map`, so a caller sent here by `lym gate redundant` sees why no map came.

`testMapRun.mapPackages(packages)` reads HEAD once, then measures each package whole in import path order: it reads HEAD before each package and, once HEAD moved, ends and counts the packages left as queued, as today.
The next package's coverage build may start while the current one's tests run, one package ahead, and `close` cancels and waits for a pending build (D2).
`testMapRun.mapPackage(job)` loses the planned-map branch: it starts from an empty map at HEAD, writes a map of file hashes with no tests for a package with no mapped test, so a package whose last mapped test was deleted gets its map rewritten with no test entries, and replaces an existing map whole, so an entry of a deleted test is gone.
`testMapRun.measureTests` loses the self-check, the probe of a kept entry and the audit count; `packageRun` drops `checked` and `probed`; `blockTable` drops `profiled`, and `packageRun.newBlockTable` no longer names the linked packages' instrumented functions.
`testMapRun.addMeasure` appends one measure per package, since no package is measured twice.
Every build and test process keeps being logged at spawn through `internal/logger` and waited on by the run, a cancelled one included, as `testcover.Toolchain` and `close` do today (`PATTERN-child-processes`).
`stampEnvironment`'s comment drops the plan's selector.
The header of `internal/gatecli/testmapstamp_test.go` drops "and the plan's selector";
no target carries it, since `edit-test-file` refuses a `_test.go` file's self glyph and the header lies outside `TestStampEnvironment`'s span, so this sentence alone carries the change.
`testmapplan.go` keeps `testMapRun.mappedTests` and `testMapRun.scanSpawns`, and its file comment says so; the file comments of `testmap.go` and `testmappackage.go` describe the map run of `lym gate test --map`, with no queue, kept entry, self-check or audit.
`testmapaudit.go`, `testmapwait.go` and `testmaporder_test.go` go whole, and `cmd/lym#unboundedWaitFiles` drops `internal/gatecli/testmapwait.go`.
The `internal/testcover` planning code is left unused here and goes on card 2.
`cmd/lym#testOnlyCodeAllowed` gains an entry for each `internal/testcover` function card 1 leaves reached by no main: `PlanUpkeep`, `NewSelector` with the `Selector` methods, `Map.FailedAt`, `Toolchain.GoRoot` and the helpers only they reach, each with the reason that card 2 deletes it, so cmd/lym's test-only-code guard passes at a batch gate between cards 1 and 2.
`gateslot.Pool.TryAcquire` goes with its test `TestTryAcquire_TakesAFreeSlotAndNeverWaits`, since the audit round of `testmapaudit.go` was its one production caller.
`testcover.FuncIndex.ProfiledFuncs` goes, since `packageRun.newBlockTable`'s self-check branch was its one production caller,
and `FuncIndex.funcsHolding` drops its `coveredOnly` parameter, since `CoveredFuncs`, its one caller left, always counts covered blocks alone;
card 2 drops its sentence from `internal/testcover/doc.go`.

Bound: `--map` skips the tiers' commands, `filter`s, `memory_mb` scopes, `timeout_min` and `test_timeout_min`, the shared build, every lint, report and guard step, the green carry and the card fixer.
What bounds it: it runs only on command, holds gate slots for its builds and tests, and writes nothing but the maps and its log.
With the build lock gone, two map runs over one package from one worktree each write its map, and the later write wins, per D8;
a failed build's write keeps the entries it read, which may come from the other run.
With the pre-walk status read gone, a package with no mapped test or only spawn-marked tests, a test or spawn scan failure and a failed coverage build are written with no dirty check, a modified or untracked file included under HEAD's hashes, until card 4 adds its check before every write;
a package whose tests run keeps the dirty-owned-file check after its build.

Tests, all in the existing files:
- `TestGateTest_RefusesBeforeResolvingTheHub` gains one row per refused partner of `--map`: `--tier 0`, `--upto 1`, `--gate card`, `--guards`, `--stop-on-fail`, `--tags tier1`, `-C sub ./pkg` and `./pkg -- -run X`, each wanting `lym gate test --map`.
- `TestTestRequestWords_MainPackages` gains the rows `MapSet` and `MapUnset` over `testRequest{targets: []string{"./pkg"}}`, wanting `--map ./pkg` and `./pkg`; it keeps its name, since the hub's `language.yaml` lists no `rename_tools`.
- The helpers `runTestMapIn`, `callTestMap` and `callTestMapCounts` drop `maxPackages` and take targets; no target means `--all` in the request's scope.
  `runTestMapCLIHelper` runs `newCommand` with this test binary as `go` and the CLI's other defaults, so `startTestMapCLI` callers pass `test --wait --map`; its, `startTestMapCLI`'s and `helperRoleTestMapCLI`'s comments name `lym gate test --map`.
  `testMapOutcome` drops the two mismatch counts and gains `LeftOut`; `testMapVerbCounts` drops `Plan`.
- `TestGateTestMap` drops the `locked` package and its held lock and the `dirty` package with its untracked `dirty/extra.go`; it keeps the `notests` package, which has neither a map nor a test and so leaves the `--all` set; the subtests `CapLeavesTheRestQueued`, `HeadMoveRetriesTheFailedBuildAndMovesTheRestWhateverTheirAge`, `AChangedFunctionReMeasuresItsTestsAndAnAddedEmptyFileMovesTheMap`, `AMissOnAMapWithEntriesRebuildsItFromNothing` and `ANamedCallWaitsForABuildLockAnotherCallHoldsAndThenRebuildsThePackageWhole` go.
  The fixture's untracked, git-excluded `ignored` package gains `ignored/ignored_test.go` with one tier0 test, so the HEAD-tracked condition is what leaves it out.
  `gateYAML` sets `scoper: go`, since under the template's default `none` every `--base` and `--card` scope is whole and maps every package.
  The fixture module narrows under it: `impactset.DeriveTargets` and `impactset.ModuleMainPackages` run the real `go list`, which exits 0 over a module whose test files import the `tiergate` package its `go.mod` does not require.
  `Refusals` drops `MaxBelowOne`, runs its rows as `test --wait --map`, and gains `NotGoJSONTier0` and `NotGoJSONTier1` (each tier in turn with `counter: vstest`, wanting the counter refusal naming that tier), `ACardWithNoPlan` and `TheRetiredVerb` (`testmap`, wanting exit 1 and an error holding `unknown subcommand "testmap"`).
  `ACardWithNoPlan` runs `test --wait --map --card 1` before any subtest writes a plan, wanting exit 1 and `computeScope`'s refusal for a worktree with no plan, opening `gate test`; `Refusals` runs before `ACardScopeMapsItsCardsPackages` writes its plan, so the worktree holds none yet.
  The second call's subtest becomes the first `--all` call: alpha's entries keep `TestAlphaOne` on block 0 and `TestAlphaTwo` on blocks 0 and 1, measured side by side at tier0's parallel of 2 (D3), and broken's failed build is recorded.
  New subtests: `AMapRunReplacesAnExistingMapWhole`, where a hand-written ghost entry in outcomes' map is gone after `outcomes` is mapped again (D2); `APackageWhoseLastTestWasDeletedGetsAMapWithNoTests`, which commits a new package `emptied` with one tier0 test, maps it, deletes and commits away its test file, and maps it again to a map with no test entry (D2); `ABaseScopeMapsThePackagesItsDiffTouches`, where a commit changing steady's source and `--base HEAD~1` builds steady alone; `AnEllipsisTargetMapsEveryPackageBelowIt`, where `./...` builds every package `--all` builds; and `ACardScopeMapsItsCardsPackages`, which writes a one-card plan through `plankit.Write` into `planparser.PlanDir(prime)` targeting `steady/steady.go`, commits a `tool` package main holding one tier0 test, and wants `--card 1` to build steady alone and `--card 1 --main-packages` to build steady and tool.
  One more new subtest, `AComputedScopeTakesNoBaseFromTheGreenRecord`, sets tier0's command to `true` and runs `runTest` with `--wait --tier 0 --all`, which records tier0 whole-green at HEAD over the clean tree;
  `test --wait --map` with no scope flag then builds every package `--all` builds, since the prime has no parent branch to diff from (D1, D7);
  it asserts the built set alone, never the exit code, which the failing tests of the `--all` set decide.
  It removes the green record and restores tier0's command when it ends, so later subtests start with no green record.
  `BuildPastItsTimeoutKeepsTheMapAndIsNotRetriedAtItsHead` keeps its first half and drops the next pass.
  `NamedPackageWithNoTrackedGoFileIsSkipped` also names `notests`, and wants `ignored` under `left_out` with `no Go file tracked at HEAD` and `notests` with `no tier0 or tier1 test and no map`, and no map written for either.
- `TestGateTestMapLinkedPackages` drops `auditCall`, `callMax` and the subtests `ALinkedCommentChangeMovesTheMapToHeadWithEveryEntry`, `ALinkedBodyChangeReMeasuresOnlyTheTestsThatCoveredIt`, `ANewTestIsMeasuredAloneAndAKeptEntryTheBuildCannotPlaceIsReMeasured`, `AQueuedTestThatWritesNoProfileStillLetsTheSelfCheckRunAKeptEntry`, both audit subtests and `ASkipAfterTheTestsCountsAgainstTheCap`; `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap` keeps its skips with no cap.
  `linked` and `linked/inner` hold no test, so the fixture writes a hand-made map for each through `testcover.WriteMap` before the first call, which keeps both in the package set under D1's has-a-map rule.
  That keeps these subtests passing: `TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker` still wants `linked` and `linked/inner` built;
  `rebuiltInnerStamp`, and with it `TheTiersDigestIgnoresTheParallelCountAndFollowsTheCommand` and `TheHubEnvDigestFollowsTheHubsVariables`, still rebuilds `linked/inner`;
  `AGoEnvFailureEndsTheWalk` still reads both maps; and `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap` still wants `linked/inner` built.
- `TestGateTestMapCLI` drops `AnUnnamedCallHasNoCapUnlessMaxIsGiven`, runs its signal subtest as `test --wait --map`, and gains `TheDetachedRunnersReplayWritesTheMap`: the child runs `test --wait` followed by `testRequest{mapMode: true, targets: []string{"p0"}}.words()`, the argv the detached runner replays, and the map of p0 is written.
- `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` calls the reshaped helper (D8 from a task worktree; `TestGateTestMap` covers the prime's shared maps), its gate config sets `scoper: go`, and it gains a second call.
  A commit adds a package `other` to the `backend` module with one tier0 test, and a one-card plan written through `plankit.Write` into `planparser.PlanDir` of the worktree's anchor targets `backend/solo/solo.go` and `tools/gen.go`, a Go file outside the `backend` module.
  `--card 1` then builds solo alone, leaving `other` unbuilt and `tools` outside the module root.
  A `--base` call cannot narrow here: `impactset.DeriveTargets` falls back to whole when the worktree root holds no `go.mod`, and this worktree's module lies under `backend`.
- `TestGateRedundant` builds cov's map with `test --wait --map cov`.

Docs: `internal/gatecli/doc.go`'s package line and its `# Coverage maps` section describe `lym gate test --map` with no planning, lock, wait, `--max`, ordering, self-check or audit, and its other lines naming `lym gate testmap` name the flag.
`README.md`, `docs/overview.md`, `internal/dobbyengine/doc.go`, `internal/gateslot/doc.go` (the testmap non-tier holder leaves its list, its `Pool.TryAcquire` sentence goes, and `test_map_max_age_days` is named as bounding nothing) and `internal/gateslot/template.yaml`'s comments name `lym gate test --map`.
`pattern/PATTERN-test-speed.md`'s redundancy-run paragraph names the flag and its coverage-map section drops the planning, self-check and audit sentences.
`pattern/PATTERN-daemon-wakeups.md` drops the map build lock wait's line and names `lym gate test --map` beside the memory guard.

**ImpactSummary:** Gatecli-local: the testmap verb, its planning, audit, lock and wait code go and its measure loop is reached through `lym gate test --map`; outside gatecli only `cmd/lym`'s wait list, its test-only-code allowlist and docs change, `gateslot.Pool.TryAcquire` and `testcover.FuncIndex.ProfiledFuncs` lose their last caller and go, and `internal/testcover`'s planning is left unused for card 2, allowlisted until card 2 deletes it.
