---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/reach.go#staleLinked
    summary: MapFormatVersion 3 is written but never read, so a map an older binary wrote, fresh by file hashes, keeps its unrun unresolved entries and the miss the entry reports persists until a full map run
    fix: "In reachStep.staleLinked, after ReadPackageMap finds the map, return its linked packages and stale=true when m.Format is nil or m.Format.Version < testcover.MapFormatVersion, before calling testreach.Fresh, so the reach refresh remaps a worktree map of an older format. Add a row to the reach refresh test whose worktree map is fresh by hashes but carries a Format of Version 2 and expect the package refreshed. In _lym/landing/summary.md, replace the sentence that existing maps keep their empty unresolved entries until the package is mapped again with the refresh behaviour this gives."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/gatecli/reach.go#caveats
    summary: caveats names unseen tests of the function's own package only, so an unresolved, out-of-process, failed, timed-out or skipped test of another package that calls the function gets no caveat naming why
    fix: "Add to testreach.Reader a method that takes an import path and returns, by module-relative package and sorted, the Unseen tests of every package whose map lists that import path in its LinkedDigests (the same set StaleLinkers walks, without the staleness test). In reachStep.caveats, for each such package other than pkg, append unseenTestCaveats(thatPackage, its unseen tests), for every changed function and not only a new one. Pin the method with a row of TestReader whose linker map holds a failed entry, and the caveat with a row of TestUnseenTestCaveats or the reach integration test."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/reach_integration_test.go
    summary: the reach step's assembly of caveats has no assertion on text, so dropping the UnseenTests wiring at reach.go:458 or misnaming the tier in the unmapped-tier caveat passes every test
    fix: "In the reach integration test, replace the Caveated boolean check for tiered.Two with an exact comparison against the caveat list, including \"tests of tiered in tier2, which no map holds, may run it unseen\". Give one stand-in map of that test an entry marked unresolved, or one with OutcomeFailed, whose package holds an unreached changed function, and assert that function's exact caveat from unseenTestCaveats."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/reach.go#unseenCause
    summary: "an unresolved test whose run passed but covered nothing is named under the cause \"recorded their runs alone as covered nothing, with no coverage\", which alone puts no test in the list, instead of its unresolved call"
    fix: "In unseenCause, take the outcome branch only when test.Outcome is neither empty nor testcover.OutcomeNoCoverage, so such a test falls to causeUnresolved; update the doc comment to say so. Add a row to TestUnseenTestCaveats with an entry {Spawn: unresolved, Outcome: testcover.OutcomeNoCoverage} expecting the unresolved caveat, and a row of exactly five tests expecting all five named with no count."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/reach.go#unmappedTierNames
    summary: a test with no tiergate.Require, or of a tier0 or tier1 the hub does not declare, goes into no map yet gets no caveat, since unmappedTierNames counts only tiers outside mappedTiers
    fix: "Give unmappedTierNames the hub's declared mapped tiers, read from the step's gate config as testMapRun.mappedTests reads r.tiers, and count a test as unmapped when !test.Tiered or its tier is not declared; name an untiered test's group as tests with no tier. Add a tier0 test of the function with an untiered test and an undeclared tier, asserting the names."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testreach/reader_test.go#TestReaderIndex
    summary: no row pins that an index of another version reads as absent, which the indexVersion bump relies on so a version-1 index without the unseen key never silences the unseen-test caveats
    fix: "Add a row to TestReaderIndex whose prepare writes the index with its current stamp and \"version\": 1, then expect TestsReaching and UnseenTests(a) to equal the answer from the maps alone."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/redundancy.go:587
    summary: the judgeRuns doc says a spawn-marked entry records no coverage, while an entry marked unresolved now holds its coverage and covers nothing only because judgeRuns skips entries that are not Measured
    fix: "Replace the line with: An outcome entry and a spawn-marked one are not measured, so they cover nothing."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: the two-way door under Merge danger does not name that the diff changes formats other binaries read, the shared maps' entries under MapFormatVersion 3 and the shared index at version 2 with its unseen key
    fix: "Under ## Merge danger in _lym/landing/summary.md, add that lym binaries of other worktrees read the hub's shared maps and index, and what an older binary does with each: it skips unresolved entries through MapTest.Measured, and it reads a version-2 index as absent and rebuilds the table in memory. Keep the two-way call only with that stated, and widen the blast radius to the reach and mutation reports and lym gate reach if they are not meant under gate."
---

The diff removes the miss the board entry reports.
The shared `internal/battencli` map marks the test that calls `hubCancelDeps`, `TestCancelIntegration_BlockedRunWithNoPair`, as unresolved only, and the shared `internal/fabricengine` map does the same for `TestRemoveLocal`.
Each has no blocks.
With this diff the map run runs such a test, and `buildTable` reaches through it via `HoldsCoverage`.
The `Root-Cause:` trailer on 94bf82d0 names that defect.
Every judge still excludes the new coverage: `JudgeMap` and `JudgeAgainst` through `notJudgedReason`, the redundancy step's kept pool and the mutation plan through `Measured`, and `lym gate redundant`'s covers and mutant pool through the verdict reason.
No finding is BLOCKING.

### [MEDIUM] The map format bump has no reader, so old maps keep the miss

**Location:** `internal/gatecli/reach.go#staleLinked`

**Issue:** No reader checks `Format.Version`.
`testreach.Fresh` compares file hashes only, and both the reach refresh's `staleLinked` and mutation's `freshMaps` rely on it.
So a worktree map an older binary wrote, still fresh by hashes, is never remapped.
Its unresolved entries keep no blocks, and the function still reads as unreached.
The incident was exactly this case: minutes-old worktree maps.
The caveat then names the scan's unresolved call as the cause, when the real cause is that the map run never ran the test.

### [MEDIUM] Unseen tests of other packages get no caveat

**Location:** `internal/gatecli/reach.go#caveats`

**Issue:** `caveats` asks `reader.UnseenTests(pkg)` for the function's own package alone.
The only caveat that looks at other packages is the stale-linker one, and it applies to new functions only.
A test of a linking package can call the function and still be unseen: out of process, failed, timed out, skipped, or unresolved in an older map.
No caveat names such a test.
The board entry asks for the reason wherever the map cannot see a call.

### [MEDIUM] The step's caveat assembly has no text assertion

**Location:** `internal/gatecli/reach_integration_test.go`

**Issue:** `TestUnseenTestCaveats` calls the helper directly.
The reach integration test checks only that a caveat exists, and its stand-in maps hold no unseen test, as commit 00eb5125 says.
The reach report lists `reachStep.caveats` and `unmappedTierNames` as reached by no mapped test.

### [LOW] An unresolved test that covered nothing is named under the wrong cause

**Location:** `internal/gatecli/reach.go#unseenCause`

**Issue:** `mayRunUnseen` lists an entry with `OutcomeNoCoverage` only when the scan marks it.
`unseenCause` still names the outcome first, and the caveat reads "recorded their runs alone as covered nothing, with no coverage".
It leaves out the unresolved call, the cause that put the test in the list.

### [LOW] Untiered tests and undeclared tiers get no tier caveat

**Location:** `internal/gatecli/reach.go#unmappedTierNames`

**Issue:** `ScanTestFuncs` gives a test with no `tiergate.Require` tier 0 and `Tiered` false.
`testMapRun.mappedTests` maps only tiered tests of declared tiers.
`unmappedTierNames` counts only tiers outside `mappedTiers`, so neither kind of test yields a caveat.

### [LOW] No row pins the index version mismatch

**Location:** `internal/testreach/reader_test.go#TestReaderIndex`

**Issue:** The bump to `indexVersion` 2 keeps the unseen list correct only because `readIndex` reads an index of another version as absent.
No row of `TestReaderIndex` writes an index of another version.

### [LOW] The judgeRuns doc gives a stale reason

**Location:** `internal/gatecli/redundancy.go:587`

**Issue:** The comment says a spawn-marked entry records no coverage.
An entry marked unresolved now holds coverage.
It covers nothing only because of the `Measured` filter at line 609.

### [LOW] Merge danger names no cross-binary read

**Location:** `_lym/landing/summary.md`

**Issue:** The rubric makes a two-way call a finding when the diff changes a record format another binary reads.
The hub's shared maps and index sit under the board directory, and every worktree's binary reads them.
Older binaries handle both changes safely: `Measured` drops the new unresolved coverage, and a version-2 index reads as absent.
The summary states neither.

## Rejected

- No separate finding for a missing row on the `namedTests` cap boundary at exactly five tests: it is folded into F4's fix, since a row alone pins no behaviour a reader relies on.
- The second test-lens point, the tier caveat text, is merged into F3: one test change fixes both.

## Focus notes

- Focus 1, readers of a map entry: every judge goes through `Measured` or the verdict reason.
  `PossibleKillers` and reach go through `HoldsCoverage`, and both are advisory.
  `fillMapTests` and `writeFailed` keep `Spawn` on carried entries.
  No defect beyond F7.
- Focus 2, the map run now running unresolved tests: those are tier1 tests that already pass in the gate's tier1 run under the same tier env.
  They run under the per-test timeout and memory cap, and a failure enters as an outcome, joins `failed_tests` and makes the call exit 1, as the doc states.
  A test that reaches `lymbin` only through a call the scan does not follow would build `lym` itself, since no `gateslot.PrebuiltLymEnv` is set.
  I found no instance, so I raise no finding.
- Focus 3, versions: covered by F1 and F8.
- Focus 4, verdict and outcome combinations: F4.
- Focus 5, ways a tier1 call can still be missed: F2 covers other packages and F5 covers untiered tests and undeclared tiers.
  A package with no map, or with a failed build, leaves the function without a caveat.
  The report's unrefreshed line names that case, so I raise no finding for it.
- Focus 6: confirmed above.
- Focus 7: F3 and F6. The version bumps have no behaviour to test beyond F6, because `MapFormatVersion` has no reader (F1).
