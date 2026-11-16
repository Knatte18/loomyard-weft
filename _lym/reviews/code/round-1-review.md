---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redundant.go:257
    summary: The covering list printed for a candidate or a refuted test is the coverage pass's list, not the CoveringTests pick the mutation verdict judged it against, so a refuted test's unique mutants can sit beside covers that never ran them
    fix: Make mutationCheck.run (internal/gatecli/redundantmutants.go:95) also return each contained test's covering list, the testcover.CoveringTests pick it already computes into MutantCandidate.Covering, in the order of contained. In runRedundant print that list as Covering at redundant.go:257 (candidates) and redundant.go:260 (refuted) in place of verdict.Covering. In TestGateRedundant stop zeroing Covering for candidates and assert each test's covering list exactly, since the pick no longer depends on seconds.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redundantmutants.go:235
    summary: A covering test that is also a candidate keeps the kill it made beside other runs in the candidate pass, which runs with rerunFailed false, so a disturbed run can confirm another candidate falsely, against the documented rule that a covering kill is rerun alone
    fix: In runMutants' cover pass (redundantmutants.go:228-241), when a cover name has ran[id][name] set and kills[id][name] true, delete kills[id][name] and append its job instead of skipping it, so it goes through c.runJobs(bin, jobs, true) and the alone rerun decides the kill. Keep skipping a cover with ran[id][name] set and no kill. Add a TestJudgeMutants-independent row only if the fix adds a branch no existing row reaches.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundantmutants.go:282
    summary: The drop-and-rebuild loop of buildMutants, the design's answer to unbuildable mutants, runs in no test; the fixture builds with no mutant dropped
    fix: Add to the cov fixture in internal/gatecli/redundant_integration_test.go a source line whose constant mutant does not compile behind the generic helper, such as a function returning 5 * time.Second that a judged test and a candidate both cover, and assert in the existing ListsTheCandidates subtest that mutation.unbuildable equals the number of mutants on that line and that the listed verdicts are unchanged.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundant.go:285
    summary: No test pins that a test marked //lym:guard is not judged; deleting the guard loop of sourceCheckingTests fails no test
    fix: Add to redundantModuleFiles' cov/cov_test.go a contained test carrying a //lym:guard line above its func, covering blocks other tests cover, and add it to the want NotJudged list of TestGateRedundant with Reason testcover.ReasonSourceCheck.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundantmutants.go:95
    summary: The covering pool commit e6a52ffc introduced, every judged test but source checks and independent of the greedy pass, has no deterministic test row; TestGateRedundant zeroes Covering and reverting to verdict.Covering passes on most runs
    fix: Once F1 prints the CoveringTests pick, assert the covering lists of TestGateRedundant's candidates and refuted test exactly, and make sure the fixture holds a candidate whose coverage-pass covers differ from its CoveringTests pick, such as one whose only killing cover is a test the greedy pass removes.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/testcover/mutant_test.go:103
    summary: TestPlanMutants, reported contained by TestMutantPlan_WriteOverlay, pins sites that test never targets (a declaring assignment, a for post statement, a const declaration, a blank assignment) and the Line and Operator fields, so it stays but lacks its keep marker
    fix: "Add the line //testtiming:keep pins the operator and line of each site and the sites the mutator leaves alone: declaring and blank assignments, for post statements and const literals directly above func TestPlanMutants at internal/testcover/mutant_test.go:103."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/sourcescan.go:31
    summary: The in-progress memo of ScanSourceReaders caches a function in a call cycle as not reaching the scan kit when the kit reference follows the call into the cycle, so a test reaching the kit through it can be judged by mutation
    fix: In ScanSourceReaders keep the in-progress set apart from the memo, and record a false answer in memo only for a function whose walk met no in-progress function; a function whose walk met one is left out of memo so a later caller walks it again. Add a row to TestScanSourceReaders_MarksTestsReachingTheScanKit with a test calling B, where A calls B and then references the kit and B calls A.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/sourcescan.go:48
    summary: A selector call on an imported package is matched against the package's own methods by name, so a test calling otherpkg.Run is marked a source reader when a same-named method of the package reaches the kit
    fix: In the *ast.SelectorExpr branch of the CallExpr case of ScanSourceReaders, skip the method lookup when callee.X is an identifier naming one of fn.imports, as spawnscan.analyzeCall does, and add a row to TestScanSourceReaders_MarksTestsReachingTheScanKit with a test calling an imported function whose name a kit-reaching method shares.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/template.yaml:4
    summary: The comments of redundancy_build_timeout_min and redundancy_test_timeout_sec, in the template and in the gateslot package doc, still name only the coverage map run and the report step, and say a test past the bound is left out, while lym gate redundant now reads both and counts a timed-out mutant run as a kill
    fix: In internal/gateslot/template.yaml lines 4 and 5 and in internal/gateslot/doc.go around lines 172-176, name lym gate redundant's mutation check among the runs each key bounds, and say that under it a mutant run past the test bound counts as a kill.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: README.md:78
    summary: The README still describes lym gate redundant as listing the tests of a package that other tests already cover, the coverage-only definition the task replaces
    fix: Change README.md line 78 to say that lym gate redundant lists from the maps the tests of a package that other tests already cover, confirmed by mutation, matching docs/overview.md.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundantmutants.go:251
    summary: The mutant run bound (ten times the unmutated seconds, at least the floor, within redundancy_test_timeout_sec) and a timed-out run counting as a kill have no test
    fix: Add to the cov fixture a function with a loop whose for condition mutant makes it endless, covered by a candidate that checks its result, set redundancy_test_timeout_sec in the test's gate config so the run ends quickly, and assert that the candidate's verdict counts that mutant as killed.
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundantmutants.go:319
    summary: Secondary paths of the check have no row - a covering kill beside other runs rerun alone, a candidate failing its unmutated run reaching not_judged through runMutants, a Go file on one side of the stale-map check only, and a mutant spanning several lines in UnbuildableMutants
    fix: Add a row to TestMutantPlan_UnbuildableMutants for a mutant whose site spans two lines with the refused line its second; add to TestGateRedundant's refusal table a stale-map row whose map lacks a file the worktree holds; and add to the cov fixture a contained test that fails when run alone, asserting it in NotJudged with testcover.ReasonFailsUnmutated, unless the map records it failed, in which case drop that part.
---

### [MEDIUM] Printed covering list is not the set the mutation verdict used

**Location:** `internal/gatecli/redundant.go:257`, `internal/gatecli/redundant.go:260`

**Issue:** `runRedundant` prints `verdict.Covering`, the coverage pass's list.
For a removable test that list comes from the tests the greedy pass kept, and it can name a source-checking test.
The mutation verdict, including each refuted test's `unique_mutants`, is computed against `testcover.CoveringTests` over the judged non-source-check tests (`redundantmutants.go:131-139`).
`internal/gatecli/doc.go` and the change description both say that pick is a contained test's covering tests.
A refuted line can name a cover that never ran its mutants and omit the one that did.

### [MEDIUM] A cover that is also a candidate keeps an unconfirmed kill

**Location:** `internal/gatecli/redundantmutants.go:219`, `:222`, `:235`

**Issue:** The candidate pass runs with `rerunFailed` false and marks `ran[id][candidate]`.
The cover pass then skips any cover already marked, so a candidate that is another candidate's cover keeps a kill made beside other runs.
`JudgeMutants` reads that kill as the cover's, which can confirm the other candidate falsely.
The documented bound covers only false refutation;
the doc says a covering test's kill beside other runs is rerun alone.

### [MEDIUM] The unbuildable-mutant rebuild loop is untested

**Location:** `internal/gatecli/redundantmutants.go:282`

**Issue:** The design drops mutants the compiler refuses and rebuilds, and the measurement dropped 23.
`TestGateRedundant` asserts `Unbuildable == 0`, and `TestMutantPlan_UnbuildableMutants` tests only the mapping.
No test reaches the rebuild, the round cap or the exit on a refused line no mutant spans.

### [MEDIUM] The guard branch of the source-check exclusion is untested

**Location:** `internal/gatecli/redundant.go:285`

**Issue:** `sourceCheckingTests` merges `impactset.FindGuardTests` names into the source readers.
The fixture holds no `//lym:guard` test, so removing that loop fails nothing, though the docs and PATTERN-test-speed promise the exclusion.

### [MEDIUM] The covering pool has no deterministic test

**Location:** `internal/gatecli/redundantmutants.go:95`

**Issue:** Commit `e6a52ffc` moved the cover pick to every judged test but source checks.
`TestGateRedundant` zeroes `Covering` because it varies by run, and no fixture test distinguishes the two pools on every run.

### [LOW] TestPlanMutants lacks its keep marker

**Location:** `internal/testcover/mutant_test.go:103`

**Issue:** The redundancy report lists it as contained, covered by `TestMutantPlan_WriteOverlay`.
Its assertions reach what that test does not: the `Line` and `Operator` fields, and the `blockCountHead` and `blockCountTail` rows, which pin sites the mutator must leave alone.
`TestMutantPlan_WriteOverlay` targets neither block.
It stays, so it carries `//testtiming:keep` naming what it pins.

### [LOW] Source scan memo misses a kit reached through a call cycle

**Location:** `internal/testcover/sourcescan.go:31`

**Issue:** `memo[fn] = false` marks a function in progress.
When A calls B, B calls A, and A references the kit after that call, B reads A's in-progress `false` and keeps `false` for good.
A test calling B is then judged by mutation, which can confirm it falsely, as the research report found for source scans.

### [LOW] Source scan follows imported selector calls into the package's methods

**Location:** `internal/testcover/sourcescan.go:48`

**Issue:** Every selector call is looked up in `src.methods` by its selector name, including a call on an imported package.
A test calling `otherpkg.Run` is marked a source reader when a same-named method of the package reaches the kit, and is wrongly left unjudged.
`spawnscan.analyzeCall` skips import selectors.

### [LOW] Timeout key docs omit the verb that now reads them

**Location:** `internal/gateslot/template.yaml:4`, `internal/gateslot/doc.go:172`

**Issue:** Both keys' comments name only the coverage map run and the report step, and the test bound says a test past it is left out.
`lym gate redundant` now reads both, and there a timed-out mutant run counts as a kill.

### [LOW] README describes the old coverage-only verb

**Location:** `README.md:78`

**Issue:** The README still says the verb lists the tests other tests already cover.
`docs/overview.md` was updated to say they are confirmed by mutation.

### [LOW] Mutant run bound and timeout-as-kill are untested

**Location:** `internal/gatecli/redundantmutants.go:251`

**Issue:** No test exercises `mutantTimeout`'s factor, floor or cap, nor a timed-out mutant run read as a kill.

### [LOW] Secondary check paths without a row

**Location:** `internal/gatecli/redundantmutants.go:319`

**Issue:** Several paths run in no test: a covering kill beside other runs that is rerun alone, a candidate failing unmutated reaching `not_judged` through `runMutants`, a stale map lacking a file the worktree holds, and a multi-line mutant site in `UnbuildableMutants`.

## Rejected

- code-tests F2, five new refusals with no reaching test: PATTERN-refusal-way-forward binds only the webster, shed and loom packages, and `internal/gatecli` is not among them.
- code-tests F8, the inherited-slot, `buildLymFor`, `ProfileBlocks` failure, plain `BuildMutantBinary` error and signal-exit items: each relays a failure through shared helpers whose own paths are tested, so no row is owed. The remaining items of that finding are kept as F12.
- code-correctness F5, mixed units in `mutantTimeout`: the factor scales the binary's reported elapsed real seconds, so `time.Second` is the right unit there, and the floor is a test-shortenable constant.
