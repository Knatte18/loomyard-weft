```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke, and file a finding where there is none.

**Behaviour → pinning test**
1. `ChangedTests` treats a method as added or changed by its text, and handles moved, base-only, overloaded, cross-class, abstract, nested, non-class and ordering cases → `TestChangedTests`, one row per case.
2. `ChangedTest.FilterTerm` and `ChangedTest.Label` → `TestChangedTest_FilterTermAndLabel`.
3. `filterTerm` refactored onto `qualifiedName` → `TestNarrow` and `TestDeriveCards` (pre-existing, reached per the reach report).
4. `parseSource` records the text of each test method:
   - block body, stacked attribute lists, expression body, generic method, bodiless method, property and field skipped, nested types → `TestParseSource_TestMethods`.
   - the `TestMethods` field on existing types → `TestParseSource` (rows A–F).
5. An expression body ends at the first depth-0 `;` (`methodEnd` → `statementEnd(i+2)`) → `ExpressionBody_EndsAtSemicolon`, but no row starts the body with a bracket. Survivor line 233 confirms the gap (F2).
6. `endMember` on `{` (survivor line 118) and in `typeDecl` (survivor line 425) → none. Both clear state that only an unrealistic input (a test-method attribute directly before `{` or before a nested type) would leak. No finding.
7. `gateslot.TierFailureNames`: reads names once each, includes guard runs, excludes failures outside any test and cause lines, stops at the cut line, returns nothing without a report → `TestTierFailureNames`. Its documented rule that a failure under a non-tier run names none has no row (F4).
8. `gitrepo.WriteTreeAtRevision` writes commit files and directories, skips files absent at the commit, refuses an invalid SHA → `TestHistoryReads`, row "WriteTreeAtRevision writes every file…". The 0o755 executable mode and the skipping of symlinks and submodules have no row (F5).
9. `fabric.ExportCodeTreeAt` writes the commit's files, not the working tree's, and refuses an invalid name → `TestCodeReadsAtCommit/ExportCodeTreeAtWritesTheCommitsFiles`.
10. `scopeInvocations` carries the request's terms onto explicit targets and marks the invocation partial → `TestPlanTierRuns`, row "explicit targets run under the call's class filter terms, partial".
11. `runTest` switches `request.root` to `checkoutRoot` after `openGreen`, so the shared build, tier dirs, stray-file check and records run in the checkout → only `TestGateRedFirst_Dotnet`, which is tier1 and skips without `dotnet` or the NUnit packages (F1).
12. `runRedFirst` dispatches to the dotnet path and skips `redundancyModule` for dotnet → only `TestGateRedFirst_Dotnet`, which skips without `dotnet` (part of F1). The Go and `none` paths stay pinned by `TestGateRedFirst` and `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged`.
13. `redFirstNoTest` extracted → Go: `TestGateRedFirst` (`redfirst_integration_test.go:89`). Dotnet: row `ADiffThatChangesNoTestMethodExitsWithTheNoTestCode`.
14. `redFirstVerdictExit` extracted, with the extra envelope keys → Go: `TestRedFirstCallRun_JudgesTheTierRunsOfTheChangedTestsOnTheBase`. Dotnet: the red, green, build-error and judged-none rows of `TestDotnetRedFirstCallRun_…`.
15. `dotnetRedFirstCall.run` has a row for each of: red, green, build error, judged none, no test, all abstract, no `{targets}` tier, no `.csproj`, two `.csproj` files. The tier request (targets, terms, `noCarry`, tiers) and the checkout contents are asserted in the `runTiers` stub. Uncovered:
    - Dropping abstract tests while running the rest → none (F6).
    - More than one test project, through `byProject` grouping, one run per project, and red and unjudged accumulating across groups → none (F3).
    - Failures of `exportBase`, `keepHeadProjectFiles` or `MkdirTemp` → none (F7).
    - A signal cancelling the run → none. The Go path has the same gap, so no finding.
16. `keepHeadProjectFiles`:
    - writes head text for changed files of the test project and keeps base text elsewhere → the stub's checkout assertions.
    - removes a file the working tree deleted → none (F8).
    - the `kept` count only feeds an informational stderr line, so survivor line 212 is no finding.
17. `filterTerms` sorts and deduplicates → none. Survivor line 327 (dropping `slices.Sort`) shows it, since every row has a single test (F2).
18. `judgeDotnetRedFirst`, `failedNameMatches` and `projectCompileError` → `TestJudgeDotnetRedFirst_…`, with rows for bare, parameterized and fully qualified names, another test's failure, a compile error inside and outside the project, and no report.
19. `nearestProject`: none found, two found → the two unjudged rows of `TestDotnetRedFirstCallRun_…`. Case-insensitive `.csproj` → none (trivial, no finding).
20. The `loomshed` log text and the wording in docs, stencil and recipe → exempt as a log field or static prose.

**Findings**

- **F1 — MEDIUM, design (coverage gap).** Location: `internal/gatecli/run.go:108-110`.
  No test that runs without a .NET SDK checks that `checkoutRoot` replaces the root for the build dir, tier dirs and run records.
  The unit test stubs `runTiers`, so `runTest` never runs with `checkoutRoot` set. `TestGateRedFirst_Dotnet` (`redfirstdotnet_integration_test.go:36-38`, and the skip in `dotnetFixtureFiles`) skips whenever `dotnet` is missing. The reach report shows `runTest` reached only by `TestGateTest_RefusesBeforeResolvingTheHub`.
  Fix: a tier1 row that sets `checkoutRoot` and runs a shell tier command and build printing their directory (for example `pwd`), asserting both are the checkout.

- **F2 — LOW, design (coverage gap).** Locations: `internal/dotnetimpact/source.go:233` and `internal/gatecli/redfirstdotnet.go:327`. Two mutation survivors point at doc-stated contracts that no row pins:
  - `methodEnd`'s `statementEnd(i+2)`: no `TestParseSource_TestMethods` row starts an expression body with `(`, `[` or `{`. With `i+3` the opening bracket is skipped, depth goes negative, and the text runs to the end of the file. Add a row such as `[Test] void M() =&gt; (Run()).Check(); void N() { }`.
  - `filterTerms` ("sorted and without repeats"): every row has one test. Add a row with two changed overloads of one method, or two methods given out of order, and assert `request.terms` in the stub.

- **F3 — MEDIUM, design (coverage gap).** Location: `internal/gatecli/redfirstdotnet.go:189-205` (`byProject`) and the per-group loop at `:113-138`.
  Running once per test project is a goal point ("by test project"), but every row of `TestDotnetRedFirstCallRun_…` has one test project.
  A mutant that puts all tests in one group, or keeps only the last group's red, passes. Add a row with changed tests in two test projects. Assert one stub call per project, each with its own target and terms, and red accumulating across both.

- **F4 — LOW, design (coverage gap).** Location: `internal/gateslot/runlog.go:471` (the `underTier &amp;&amp;` condition).
  The doc says "a failure under any other run names none", but no row of `TestTierFailureNames` puts a `  &lt;name&gt;:` line under a non-tier run such as a lint step or `build`. Row 2's build block holds only `its output:` and a 4-space line, both excluded on other grounds. Add such a line under a `build (exit 1)` or `lint …` header and expect it unread.

- **F5 — LOW, design (coverage gap).** Location: `internal/gitrepo/blobread.go:228-231,252-255`.
  `WriteTreeAtRevision` documents the 0o755 mode for executables and skips symlinks and submodules. No assertion pins either, and a base checkout that drops an executable bit can break a build script. Extend the `TestHistoryReads` row with an executable file and a symlink in the commit.

- **F6 — LOW, design (coverage gap).** Location: `internal/gatecli/redfirstdotnet.go:72`.
  No row mixes abstract and concrete changed tests, so nothing checks that only the concrete ones reach `{filter}` and the labels. Add a head with one abstract-class test and one concrete-class test, and assert that the terms hold only the concrete one.

- **F7 — LOW, design (coverage gap).** Location: `internal/gatecli/redfirstdotnet.go:88-100`.
  The failure paths of `exportBase` and `keepHeadProjectFiles` (exit 1 with a way forward) have no row. Add a row whose `exportBase` stub returns an error, and assert exit 1, the "cannot check out the base" text and that the tiers did not run.

- **F8 — LOW, design (coverage gap).** Location: `internal/gatecli/redfirstdotnet.go:221-224`.
  `keepHeadProjectFiles` removes a test-project file the working tree deleted, and no row covers it. Add a changed path absent from the working tree, for example a deleted `Calc.Tests/old.json` present at the base, and assert it is gone from the checkout.

- **F9 — LOW, consistency (test economy).** Location: `internal/gatecli/redfirstdotnet_test.go:23`.
  The redundancy report lists `TestJudgeDotnetRedFirst_ReadsTheRunsExitCodeFailureReportAndCompileErrors` as contained in `TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase`. Its assertions differ:
  - it pins parameterized and fully qualified name matching, a compile error outside the project being unjudged, another test's failure being unjudged, and the exact `buildError` text;
  - the call test asserts only a bare-name red, a `CS0117` substring and a `restore failed` exit 1.

  Keep it, and mark it `//testtiming:keep pins parameterized and fully qualified failure-name matching, an out-of-project compile error judging none, and the exact build-error text, which TestDotnetRedFirstCallRun_… never asserts` directly above its `func Test…` line.

Tool calls:
- grep of the gatecli and loomshed tests for `RedFirstUnjudgedExit`, `RedFirstNoTestExit`, `checkoutRoot` and `testtiming:keep`, plus the red-first test function list. My context lacked whether the Go no-test and unjudged paths, and `checkoutRoot`, had any pinning test outside the diff.
