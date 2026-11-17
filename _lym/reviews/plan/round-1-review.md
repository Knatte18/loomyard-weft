---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/01-gate-test-map-mode.md:149
    summary: Card 1's package set drops linked and linked/inner from TestGateTestMapLinkedPackages, whose subtests card 1 leaves expecting their maps
    fix: >-
      In card 1's TestGateTestMapLinkedPackages bullet, say how the fixture keeps linked and linked/inner in the package set under D1's has-a-map rule.
      The fixture writes a hand-made map for each of them through testcover.WriteMap before the first call.
      State the subtests this keeps passing: TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker still wants linked and linked/inner built;
      rebuiltInnerStamp, and with it TheTiersDigestIgnoresTheParallelCountAndFollowsTheCommand and TheHubEnvDigestFollowsTheHubsVariables, still rebuilds linked/inner;
      AGoEnvFailureEndsTheWalk still reads both maps; and ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap still wants linked/inner built.
      Alternatively, rewrite each of those subtests over a package that holds a tier0 test, and say which one.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/01-gate-test-map-mode.md:146
    summary: Card 1's scope-narrowing subtests never narrow, since the fixtures' gate config sets no scoper and the template default none makes every non-explicit scope whole
    fix: >-
      In card 1's Tests, have TestGateTestMap's gateYAML and TestGateTestMapInATaskWorktreeKeepsItsOwnMaps's gate config set scoper: go.
      Then record in the card whether impactset.DeriveTargets narrows over the stand-in fixture module.
      It runs the real go list, and the fixture's test files import the loomyard tiergate package, which the fixture's go.mod does not require.
      If go list fails there, the derivation falls back to whole with a nil error.
      In that case ABaseScopeMapsThePackagesItsDiffTouches and the task-worktree --base HEAD~1 call cannot narrow.
      Replace them with a --card case, which reads its packages from the plan with no go list, or make the fixture module resolvable.
      Also give the task-worktree --base call a second package that the diff leaves out, so the call pins the narrowing.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-map-run-tier-bounds.md:107
    summary: Card 3's ATestTheGuardKillsBesideAnotherPassesAloneAndIsListed needs a guard kill that cannot happen, since memory_quit_mb bounds each test binary's own tree
    fix: >-
      Drop the subtest ATestTheGuardKillsBesideAnotherPassesAloneAndIsListed from card 3's Tests, with its two Crowded HoldsMemory tests.
      TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone, with its Collides tests, keeps D5's rerun-alone-and-list path covered.
      ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn keeps the guard's kill path covered.
      Add one Intent sentence: the pressure trigger, which can kill a test for its siblings' memory, needs a 1 GiB tree under a cgroup limit, so no test drives it.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/01-gate-test-map-mode.md:144
    summary: Card 1's Refusals row ABadBase cannot reach a computeScope refusal, since a non-ancestor --base falls back to a whole scope with no error
    fix: >-
      Replace card 1's ABadBase row with ACardWithNoPlan.
      It runs test --wait --map --card 1 before any subtest writes a plan, wanting exit 1 and the computeScope refusal for a worktree with no plan, opening gate test.
      Refusals runs before ACardScopeMapsItsCardsPackages writes its plan, so the worktree holds none yet.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/01-gate-test-map-mode.md:116
    summary: Card 1's package-set rule leaves a whole scope or tier share from a computed, base or card call unhandled, so a fallback to whole yields no packages
    fix: >-
      In card 1's testMapRun.packages bullet, replace "a whole scope (--all) is every package modulePackages finds" with a rule that covers every whole case.
      A scope whose whole is set, from --all, a scoper other than go or a computed fallback, is every package modulePackages finds.
      So is a scope where tier0's or tier1's share from scope.forTier has whole set.
      Otherwise the set is the union of the two shares' module targets.
      Also note that keptTiers derives the shares from the request's tiers, so a map call with no --tier gets tier0's share alone.
      State whether tier1's forTier fallback contributes nothing, and that this is the intended set.
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/01-gate-test-map-mode.md:133
    summary: Card 1 removes the per-package build lock and the pre-walk dirty-file skip without stating what they now let through or what bounds it
    fix: >-
      Extend card 1's Bound paragraph with two sentences.
      First, with the build lock gone, two map runs over one package from one worktree each write their map at once and the later write wins, per D8.
      A failed build's write keeps the entries it read, which may come from the other run.
      Second, with the pre-walk status read gone, a package with no mapped test and a modified or untracked file is written under HEAD's hashes until card 4 adds its check before the write.
      Every other package keeps the dirty-owned-file check after its build.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/01-gate-test-map-mode.md:131
    summary: Card 1 leaves testcover's planning code unreachable from a main, so cmd/lym's test-only-code guard fails at any batch gate between cards 1 and 2
    fix: >-
      Add `cmd/lym#testOnlyCodeAllowed` under card 1's Edit, with one Intent sentence.
      It adds an allowlist entry for each testcover function card 1 leaves unreachable: PlanUpkeep, NewSelector with the Selector methods, Map.FailedAt, Toolchain.GoRoot and their helpers.
      Each entry's reason is that card 2 deletes the function.
      Add `cmd/lym#testOnlyCodeAllowed` under card 2's Edit, with an Intent sentence saying card 2 removes those entries, since an entry that matches no unreachable function fails the guard.
      validate-plan --overlay admits both target additions.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-map-run-tier-bounds.md:98
    summary: Card 3 lists the subtests that exit 1 after its change but misses card 1's AMapRunReplacesAnExistingMapWhole and AnEllipsisTargetMapsEveryPackageBelowIt
    fix: >-
      Under card 3's TestGateTestMap bullet, add that AMapRunReplacesAnExistingMapWhole wants exit 1, since outcomes' TestOutcomesFails fails.
      Also add that AnEllipsisTargetMapsEveryPackageBelowIt wants exit 1, since ./... holds outcomes, alpha's TestAlphaHangs and broken's failed build.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-map-run-tier-bounds.md:90
    summary: Card 3 states TestMapSideBySide and TestMapSlotAsk are tier0 but names no file for them, and the map tests' file is tier1
    fix: >-
      In card 3's Tests, name the tier0 file the two tests go in, such as internal/gatecli/testmapstamp_test.go, whose tests are tier0.
      Never use testmap_integration_test.go, whose tests are tier1.
  - id: F10
    severity: LOW
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/02-testcover-planning-delete.md:25
    summary: Card 2 deletes Toolchain.GoRoot but its doc.go edit leaves the sentence naming GoRoot at internal/testcover/doc.go line 68
    fix: >-
      In card 2's doc.go sentence, add that the GoEnvironment line drops "and GoRoot its root directory".
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/01-gate-test-map-mode.md:120
    summary: Card 1's left_out reason for a package with no tier0 or tier1 test and no map has no named test that reaches it
    fix: >-
      In card 1's TestGateTestMap bullet, keep the notests package in the fixture.
      Have NamedPackageWithNoTrackedGoFileIsSkipped also name notests and want it under left_out with the reason no tier0 or tier1 test and no map, and no map written.
  - id: F12
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-map-run-tier-bounds.md:96
    summary: Card 3's Collides stand-in passes alone only after a 2-second wait under the fixture's 3-second test bound, and it never says whether it removes its record
    fix: >-
      Specify the Collides suffix as runsApart does: it records itself, removes its own record when it ends, and fails as soon as another record appears.
      Its wait must sit far below the fixture's per-test bound of 20 times 150 ms, such as the existing testMapExcludeWait of 300 ms, or the bound must be raised for those subtests.
      A loaded machine then cannot turn an alone pass into a timeout.
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-map-run-clean-checks.md:44
    summary: Card 4 guards the write after a failure while the tests run, but no named test reaches that path
    fix: >-
      Either name a test that reaches it, a runBatch reduction failure over a dirty linked file wanting no map written, with the input that makes the reduction fail.
      Or state in card 4 that this path shares checkClean's call inside writeFailed with the failed-build path AFailedBuildOverADirtyLinkedFileWritesNoMap reaches, and that no separate input reaches it.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-gate-test-map-mode.md:128
    summary: Card 1 drops the plan's selector from stampEnvironment's comment but leaves internal/gatecli/testmapstamp_test.go's header naming it
    fix: >-
      Add one Intent sentence to card 1: the header of internal/gatecli/testmapstamp_test.go drops "and the plan's selector".
      No target can be added for it.
      The file self glyph of a _test.go file is refused by edit-test-file, and the header lies outside TestStampEnvironment's span, so the sentence alone carries the change.
---

### [BLOCKING] Card 1 leaves TestGateTestMapLinkedPackages depending on maps the new package set never builds

**Location:** `_lym/plan/01-gate-test-map-mode.md:149`

**Issue:** D1 puts a package in the set only when it has a map or holds a tier0 or tier1 test.
In the linked fixture, `linked` and `linked/inner` hold no test file (`testmap_integration_test.go:1372-1378`) and have no map in a fresh hub.
So they leave the set.
Several subtests that card 1 keeps still depend on those maps:

- `TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker` wants both built (line 1445).
- `rebuiltInnerStamp` wants `linked/inner` built (line 1435), and both digest subtests use it.
- `AGoEnvFailureEndsTheWalk` reads both maps (lines 1867-1868).
- `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap` wants `linked/inner` built (line 1717), and card 4 restates that.

Card 1 handled the same change for `notests` in `TestGateTestMap` but not here, so card 1's tier1 batch gate fails.
The affected-set lens raised this, and the handler confirmed it against the fixture and card 1's package-set rule (lines 119-120).

### [BLOCKING] Card 1's scope-narrowing subtests run under the none scoper and cannot narrow

**Location:** `_lym/plan/01-gate-test-map-mode.md:146`

**Issue:** The fixture gate configs set no `scoper` (`testmap_integration_test.go:668-669, 725-728`).
`internal/gateslot/template.yaml:40` defaults it to `none`, and `redundancy_integration_test.go:62` shows that tests needing narrowing set `scoper: go` explicitly.
Under `none`, `computeScope` returns a whole scope for the `--base`, computed and `--card` forms (`scope.go:186-188`).
So these cases map every package, and they fail or pin nothing:

- `ABaseScopeMapsThePackagesItsDiffTouches`;
- `ACardScopeMapsItsCardsPackages`, both with and without `--main-packages`;
- the task-worktree `--base HEAD~1` call, whose module has the one package `solo`.

Setting `scoper: go` is not enough on its own.
`impactset.DeriveTargets` lists packages with the real `go list` (`internal/impactset/targets.go:72-77`) and falls back to whole with a nil error when that fails.
The fixture's test files import the loomyard `tiergate` package, which the fixture module's `go.mod` does not require.
The card leaves to the implementer the fact of whether narrowing can happen over this fixture at all, which is a research step the rubric bars.
The guard-reach lens raised the scoper half; the handler added the `go list` half from `targets.go`.

### [MEDIUM] Card 3's guard-kill-beside-another subtest asks for a kill the guard never makes

**Location:** `_lym/plan/03-map-run-tier-bounds.md:107`

**Issue:** The memory guard kills a test binary whose own tree holds more than `memory_quit_mb` (`memoryguard.go:38-42, 96-99`).
Its pressure trigger kills only a tree of at least `pressureKillFloor`, 1 GiB, and only under a cgroup limit (`memoryguard.go:27-33`).
With `memory_quit_mb` set above one 384 MB hoard and below two, neither process passes the limit, so nothing is killed and `passed_alone` stays empty.
The guard-reach lens filed this as BLOCKING.
The handler rates it MEDIUM: D5's rerun-alone path stays covered by the Collides subtest, and the fix only cuts the row.

### [MEDIUM] Card 1's ABadBase refusal row has no input that reaches a refusal

**Location:** `_lym/plan/01-gate-test-map-mode.md:144`

**Issue:** Under `scoper: none` a `--base` scope is whole.
Under `go`, `diffBaseGroups` passes the base through (`scope.go:358-359`).
`DeriveTargets` then turns a non-ancestor base into a whole scope with a nil error (`internal/impactset/targets.go:63-65`).
So `--base nosuch` never makes `computeScope` fail, and the row that should pin "a `computeScope` error is the run's refusal" cannot pass.
`computeScope`'s refusals before any slot are an unknown card id and a worktree with no plan (`scope.go:128-130`).

### [MEDIUM] Card 1's package set mishandles a whole scope or share that does not come from --all

**Location:** `_lym/plan/01-gate-test-map-mode.md:116`

**Issue:** Card 1 treats "a whole scope (`--all`)" as every package and every other scope as the union of tier0's and tier1's shares' module targets.
`computeScope` also returns a whole scope in other cases:

- under the `none` or `dotnet` scoper (`scope.go:186-188`);
- for a computed scope that narrowed nothing (`scope.go:271-273`);
- with a tier share whose `whole` is set inside `tiers`, when a tier has no green record or merge base (`scope.go:268-276`).

A whole share has no module targets, so the union as written yields no packages for it.
D1 says the scope is chosen as for any call, and the diff from the merge base is a named form.
A map call from a worktree with no parent branch would map nothing.
The card also never says which tiers' shares exist: `diffBaseGroups` takes them from `keptTiers(cfg, request)`, and a map request names no tier.

### [MEDIUM] Card 1 removes the build lock and the pre-walk dirty check with no stated bound

**Location:** `_lym/plan/01-gate-test-map-mode.md:133`

**Issue:** The rubric's attack-surface rule wants a card that removes a guard to state what now passes and what bounds it.
Card 1's Bound paragraph covers only what `--map` skips of the tier machinery.
It deletes the per-package build lock (`testMapRun.lockPackage`, `testmapwait.go`) and the pre-walk status read behind `testMapRun.dirtyGoFile`, and states neither change.
With the lock gone, two map runs over one package race, and the later write wins, as D8 records.
With the status read gone, a package with no mapped test writes its map with no dirty check (`testmappackage.go:131-132`) until card 4 adds `checkClean` before every write.

### [MEDIUM] Card 1 strands testcover's planning code as unreachable until card 2 lands

**Location:** `_lym/plan/01-gate-test-map-mode.md:131`

**Issue:** Card 1 removes the last production callers of `testcover.PlanUpkeep`, `NewSelector`, `Map.FailedAt` and `Toolchain.GoRoot` (`testmapplan.go:68, 88, 133, 141`; `testmapaudit.go:37`).
Card 2 deletes those functions, as the plan spec's separate-delete-card rule asks.
`cmd/lym/testonlycode_linux_test.go:80-94` is a tier1 guard test, and it fails on a production function no main reaches.
So a batch gate, which runs tier0 and tier1 with the guard step, fails if a batch boundary falls between cards 1 and 2, and nothing in the plan rules that out.
The handler checked the combined allowlist fix with `lym loom validate-plan --overlay`: it reports no blocking finding, and neither does the `{}` baseline.

### [LOW] Card 3 misses two of card 1's subtests that exit 1 after its change

**Location:** `_lym/plan/03-map-run-tier-bounds.md:98`

**Issue:** Card 3 makes a failed or timed-out entry exit 1 and lists the subtests that must assert it.
It omits card 1's `AMapRunReplacesAnExistingMapWhole`, which maps `outcomes` with its failing test, and `AnEllipsisTargetMapsEveryPackageBelowIt`, whose `./...` holds `outcomes`, `alpha`'s hanging test and `broken`.

### [LOW] Card 3 names no file for its two tier0 tests

**Location:** `_lym/plan/03-map-run-tier-bounds.md:90`

**Issue:** `TestMapSideBySide` and `TestMapSlotAsk` are `plan:` handles with no file.
Every test of one file names one tier (`PATTERN-test-speed`, enforced by `cmd/lym/tiertags_test.go`).
`testmap_integration_test.go`, where the map run's tests live, is tier1 (lines 4 and 715).

### [LOW] Card 2 leaves testcover's doc naming the deleted GoRoot

**Location:** `_lym/plan/02-testcover-planning-delete.md:25`

**Issue:** `internal/testcover/doc.go:68` reads "GoEnvironment reads the toolchain's Go version and platform, and GoRoot its root directory".
Card 2 deletes `Toolchain.GoRoot`, but its doc edit names only the `PlanUpkeep` and `Selector` paragraphs and the `Map.FailedAt` sentence.
Both the affected-set and repo-invariants lenses raised this.

### [LOW] No named test reaches card 1's second left_out reason

**Location:** `_lym/plan/01-gate-test-map-mode.md:120`

**Issue:** Card 1 drops the `notests` package, which has neither a test nor a map, from the fixture (line 142).
Its only `left_out` assertion names `ignored`, with the reason `no Go file tracked at HEAD` (line 148).
So the reason `no tier0 or tier1 test and no map` is prescribed with no test that reaches it.

### [LOW] Card 3's Collides stand-in leans on a timing margin a loaded machine can cross

**Location:** `_lym/plan/03-map-run-tier-bounds.md:96`

**Issue:** An alone Collides run passes only once `testMapCollideWait`, 2 s, ends.
The fixture bounds each test at 20 times 150 ms, which is 3 s (`testmap_integration_test.go:589, 721`), so the margin is one second under load.
`PATTERN-test-speed` bars a test whose outcome turns on a time bound a loaded machine can reach.
The card also never says whether a Collides test removes its record when it ends, as `runsApart` does (line 411).
If records stay, a test's alone rerun sees its sibling's record and fails.

### [LOW] Card 4's guarded write after a failure while the tests run has no reaching test

**Location:** `_lym/plan/04-map-run-clean-checks.md:44`

**Issue:** Card 4 lists four guarded write paths.
Its new subtests reach the no-build, scan-failure and failed-build paths (lines 61-69).
None reaches the path where `measureTests` takes `writeFailed` on a `runBatch` reduction error (`testmappackage.go:278-283`).

### [NIT] Card 1 leaves a stale selector mention in testmapstamp_test.go's header

**Location:** `_lym/plan/01-gate-test-map-mode.md:128`

**Issue:** `internal/gatecli/testmapstamp_test.go:1` calls the stamp one that "a coverage map build and the plan's selector both compute".
Card 1 deletes the selector and drops it from `stampEnvironment`'s comment, but no card names this header.
`edit-test-file` refuses the file's self glyph as a target, so no admissible target exists for the change; the card's Intent carries it.
Both the affected-set and repo-invariants lenses raised this.

## Rejected

- Test economy of card 3's `TestMapSideBySide` and `TestMapSlotAsk` (a focus question): not a defect.
  Each pins cases no integration subtest reaches: `Parallel` 0 means `runtime.NumCPU()`, a capped count below the memory quotient wins, and tier0 1 beside tier1 2 gives 2.
  The integration subtests `ATiersMemoryMBClampsItsTestsToOneAtATime` and `AMapRunHoldsTheLargerSlotAskOfItsTiers` pin that the run applies the helpers, which the tables cannot.
- Card 1's size, and its deleting gatecli helpers on the same card that removes their last caller: not flagged.
  The deleted symbols are private planning helpers that the reshaped loop's own edit drops, so they belong to that unit under the granularity rule's private-helper clause.
  The testcover deletes, which serve another package, are on their own card.

## Notes on the focus directives

- Card order, cards 1 to 4: held, apart from F7's guard.
  `dirtyOwnedFile`, `readHead`, `readPackageTree`, `testMapRun.skip` and `endJob` survive cards 1 to 3.
  `changedFiles` stays until card 4 deletes it with its caller's rewrite.
  After card 1, nothing outside `internal/testcover` reaches card 2's deletes.
- Edit-card shape: every re-signed member carries its arrow (`packages`, `mapPackages`, `mapPackage`, `runBatch`, `runSideBySide`, `write`, `writeFailed`, and the test helpers).
  `measureTests`, `addMeasure` and `testMapSkip` keep their signatures, or are structs and take no arrow.
  `prebuild.go` and both `doc.go` edits change free-standing comments, so their file self glyphs are right.
  No card puts a rewritten ref under `Extends`.
- Verify lines: no card carries a `**Verify:**` line naming a raw test command.
- Board scope: no card edits this task's board entry (`T23`), files a task or promotes a note.
