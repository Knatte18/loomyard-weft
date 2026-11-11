---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmappackage.go:421
    summary: stampEnvironment and mappedTierConfigs have no test, so the D6 criteria on a parallel change and an env.yaml change are unpinned
    fix: Add a table test in internal/gatecli (external package, t.Parallel) over stampEnvironment with rows for a changed tier command, a changed tier env value, a changed tier parallel value (the digest stays equal), a changed hub env.yaml variable (HubEnvDigest differs), and an unchanged input (equal). Assert on TiersDigest and HubEnvDigest exactly. Expose the unexported function through export_test.go if the test is in package gatecli_test, and add a row for mappedTierConfigs returning only the tiers the config declares.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/declcompare.go:175
    summary: compareTestSources misses on any import difference, so a new test file importing testing misses and its new Test functions are never selected
    fix: In compareTestSources, do not miss on imports when one side of the file is absent, and for a file present on both sides accept an added or removed import of a standard-library or linked package, as compareGoSources does with importAllowed. Hoist that import judgment into packageChanges, which holds the map and the selector, by returning the added and removed import paths in testFileComparison. Keep the miss for an import outside both sets. Change the OwnTestFileWithOnlyTestFunctionsAdded fixture to import testing, and add a row for a removed test file that imported testing (no miss, no tests selected).
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testcover/testselect_test.go:288
    summary: The environment-stamp miss rows cover only the Go version, go.mod and go.sum; goos, goarch, tiers digest and hub env digest have no row and the miss detail is not checked
    fix: Add rows to TestSelectorSelect that edit m.Format.Environment.GOOS, GOARCH, TiersDigest and HubEnvDigest, each with wantMiss MissEnvironment. Add a wantDetail field to the table and assert on one row that the detail names the differing field, for example "the map's stamp differs in tiers_digest".
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: internal/testcover/testselect.go:217
    summary: mapMiss still misses on BuildFailure even when the map holds test entries, though D2 says a build-failure record beside test entries is no miss
    fix: In mapMiss, miss on BuildFailure only when len(m.Tests) == 0, so a map that records a failure beside passing entries stays usable. Add a TestSelectorSelect row with BuildFailure set and test entries kept, expecting no miss, and keep the BuildFailure row for a map with no tests. If the parent reads D5's failure field as a different field from Map.BuildFailure, record that with lym loom decision add and leave the code.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testcover/testselect.go:411
    summary: A commit target whose files cannot be read has no row, so the error return of Select for an unreadable target is unpinned
    fix: Add a case to TestSelectorSelect or a short separate test where Target.Commit names a revision the fakeCommits does not hold, asserting Select returns a non-nil error and no miss. Add one case for a target whose go.mod read fails, covering environmentMiss.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/coveragescope.go:296
    summary: The unreadable miss openSelector returns when the Go toolchain cannot be read is untested
    fix: Add a TestNarrowCoverage-level or openSelector-level case with a request.goBinary that fails, asserting every importing package takes a miss of reason MissMapUnreadable naming the read failure.
  - id: F7
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/testcover/declcompare.go:169
    summary: The MissTestFile return for a test file that changes its package clause has no row
    fix: Add a TestSelectorSelect row that replaces "package b" with "package b_test" in b/funcs_test.go, with wantMiss MissTestFile.
---

### [BLOCKING] Stamp builder untested against its acceptance criteria

**Location:** internal/gatecli/testmappackage.go:421
**Issue:** The record's D6 criteria say a changed tier `parallel` value changes no stamp and a changed `env.yaml` variable does.
`stampEnvironment` is the one builder both the map build and the gate scope call, and no test under `internal` references it or `mappedTierConfigs`.
A regression that put the parallel count into the digest, or dropped the env.yaml digest, would pass every test the diff adds.

### [MEDIUM] Imports make a new or removed test file a miss

**Location:** internal/testcover/declcompare.go:175
**Issue:** D2 says a test whose `Test` function is new is affected.
`compareTestSources` compares imports with `maps.Equal`, and nearly every test file imports `testing`.
A new test file therefore misses and its Test functions are never selected, and a deleted one misses too, though D2 calls a removed Test function no miss.
The change description admits the first case.
The fixture row `OwnTestFileWithOnlyTestFunctionsAdded` hides it by using `*T` with no import.

### [MEDIUM] Stamp-miss rows leave four stamp fields and the detail unpinned

**Location:** internal/testcover/testselect_test.go:288
**Issue:** `stampDifferences` compares seven fields; the rows move three.
A differing `goos`, `goarch`, `tiers_digest` or `hub_env_digest` could be dropped from the comparison unnoticed, and the row loop only checks that the detail is non-empty.

### [LOW] A build-failure record beside test entries still misses

**Location:** internal/testcover/testselect.go:217
**Issue:** D2 lists "D5's build-failure record beside test entries is no miss" among the selector's rules.
`mapMiss` misses on any non-empty `BuildFailure`, test entries or not.
The change description reads D5's field as a separate one the upkeep plan will write, and D5 is not in the record this review holds, so the reading may be right; no test pins either way.

### [LOW] An unreadable commit target has no row

**Location:** internal/testcover/testselect.go:411
**Issue:** `fakeCommits` serves `targetCommit` in every row, so the `CodeFileAt` error path of `readTargetFile`, and the error `Select` returns for it, are never run.

### [LOW] Unreadable toolchain miss in openSelector is untested

**Location:** internal/gatecli/coveragescope.go:296
**Issue:** The new `GoEnvironment` call adds a failure path that turns every importing package into a miss; no test reaches it.

### [NIT] Package-clause change in a test file has no row

**Location:** internal/testcover/declcompare.go:169
**Issue:** The `MissTestFile` branch for a changed package clause is run by no row.

## Rejected

- Lens code-correctness finding 2 (`Select` returns a nil miss with an error): `coverageSelections` turns a non-nil error into a `MissMapUnreadable` miss at `internal/gatecli/coveragescope.go:253`, so the package runs whole.
- Lens code-tests note on `testmap.go` `prepare` picking `moduleDir` from the lowest mapped tier: the old loop took the first declared tier of `mappedTiers`, tier0 then tier1, which is the lowest, so behaviour is unchanged, and existing testmap tests cover `prepare`.
- Lens code-tests notes that a removed whole test file and the `MissCommitUnreadable` return of `compareTestFile` are unpinned: folded into F2's added row and not worth a finding of their own.
- Declared PATTERN changes, `Pattern-Departure:` trailers and the change description: the directive declares no PATTERN change, neither commit carries a trailer, and the description's claims match the diff.
