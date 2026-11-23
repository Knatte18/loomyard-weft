---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gitrepo/blobread.go:226
    summary: WriteTreeAtRevision skips every executable blob, since go-git's IsRegular is false for filemode.Executable, so a 100755 file never reaches the base checkout and the 0o755 branch of writeBlobFile is dead
    fix: "In WriteTreeAtRevision replace `if !file.Mode.IsRegular()` with a check that admits filemode.Regular, filemode.Deprecated and filemode.Executable and skips everything else (symlinks and submodules). In the `WriteTreeAtRevision writes every file as the commit stores it` row of TestHistoryReads in internal/gitrepo/blobread_integration_test.go, commit an executable file and a symlink. Assert that the executable is written with its contents and an executable mode bit, and that the symlink is absent."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/gatecli/redfirst.go:135
    summary: The diff adds three new branches on the scoper literal "dotnet" (redfirst.go:135, 148, 166), which PATTERN-code-conventions "No hard-coded choice" forbids; no Pattern-Departure trailer excuses them
    fix: "Add the red-first behaviour of each scoper as a row of a table keyed by scoper name in internal/gateslot/config.go, beside scoperHasGoCoverageMaps. For example, a map that says whether a scoper runs red-first by overlay (go), by base checkout (dotnet) or not at all, exposed through a Config method. Branch redfirst.go:135, 148 and 166 on that method's answer instead of comparing cfg.Scoper with the \"dotnet\" literal."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirstdotnet.go:282
    summary: judgeDotnetRedFirst calls any exit-0 run green, even one whose filter matched no test, so a name the FullyQualifiedName~ term misses gives RedFirstGreenExit and holds the writer for a reason they cannot act on; the run exits 0 because VSTest only warns that no test matches
    fix: "Pass the run's discovered-test count, which the tier summary line and run record carry, into judgeDotnetRedFirst. Judge a run that exited 0 with no test discovered as unjudged, either redFirstCannotJudge naming the filter terms or the judged-none outcome, never redFirstGreen. Add a row to TestJudgeDotnetRedFirst_ReadsTheRunsExitCodeFailureReportAndCompileErrors and one to TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase where the tier exits 0 having run no test."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/gatecli/redfirstdotnet.go:136
    summary: The entry's "halt only when selection or build fails for a reason the writer can act on" is built only in part; a failed restore, or a build failure outside the test project, keeps the tier code as unjudgedCode and exits 1, which the red-first gate fails terminally
    fix: "In dotnetRedFirstCall.run, when judgeDotnetRedFirst returns redFirstUnjudged and the captured output shows the run failed before any test ran, end the call with redFirstCannotJudge naming the cause. Before any test ran means a restore or build failure with no failure report naming a test and no compile error under the test project. Keep exit 1 only for a run that started tests and judged none of them, such as a timeout or a crash. Update the ARunThatJudgedNoneExitsOne row of TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase, and the What exits 1 claim in _lym/landing/summary.md, to match."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirstdotnet_test.go:90
    summary: Running once per test project is a goal point, but every row of TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase has one test project, so byProject's grouping and the red and unjudged results across groups have no test
    fix: "Add a row to TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase whose diff changes a test in each of two test projects. The runTiers stub records each request. Assert one call per project, each with that project's checkout .csproj as its one target and only its own tests' terms, and assert that the envelope's red_tests holds the red tests of both projects."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:108
    summary: Only TestGateRedFirst_Dotnet reaches the checkoutRoot swap in runTest, and it skips without a .NET SDK, so nothing that runs everywhere checks that the build and tier dirs resolve in the checkout
    fix: "Add a tier1 row to an existing runTest test in internal/gatecli. Give it a tier and a shared build whose shell commands print their working directory, run it with testRequest.checkoutRoot set to a t.TempDir() copy, and assert that both printed directories lie under that checkout."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dotnetimpact/source_test.go#TestParseSource_TestMethods
    summary: "Two mutation survivors are documented contracts no row pins. One is methodEnd's statementEnd(i+2) at source.go:233: no expression body opens with a bracket. The other is filterTerms sorting and removing repeats at redfirstdotnet.go:327, where two changed overloads of one method give duplicate terms."
    fix: "Add a TestParseSource_TestMethods row whose source is `class A { [Test] void M() => (Run()).Check(); void N() { } }`, expecting Text `[Test] void M() => (Run()).Check();`. Add a TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase row whose head adds two methods out of name order plus two overloads of one of them. Assert that the stub's request.terms are sorted, with that method's term once."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirstdotnet_test.go:90
    summary: "Three branches of dotnetRedFirstCall.run have no row: running only the concrete tests when abstract-class tests are mixed in, the exit-1 refusal when exportBase or keepHeadProjectFiles fails, and keepHeadProjectFiles removing a test-project file the working tree deleted"
    fix: "Add three rows to TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase. (1) The head adds one abstract-class test and one concrete-class test; assert that request.terms hold only the concrete test's term. (2) The exportBase stub returns an error; assert exit 1, the text \"cannot check out the base\" and that runTiers never ran. (3) A changed path, Calc.Tests/old.json, is present in the base tree and absent from the working tree; assert the stub finds no such file in the checkout."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gateslot/failedtests_test.go:210
    summary: TierFailureNames' documented rule that a failure under a non-tier run names none has no row; no case puts a `  <name>:` line under a build or lint header
    fix: "Add a row to TestTierFailureNames whose report holds a `build (exit 1), log /runs/b.log:` header followed by a `  Ns.C.M:` line, then a tier0 run naming `  AreaIsPositive:`. Expect only AreaIsPositive."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/gatecli/redfirstdotnet_test.go:23
    summary: The redundancy report lists TestJudgeDotnetRedFirst_ReadsTheRunsExitCodeFailureReportAndCompileErrors as contained in TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase, but it pins what that test never asserts, so it needs a keep marker
    fix: "Directly above `func TestJudgeDotnetRedFirst_ReadsTheRunsExitCodeFailureReportAndCompileErrors`, add `//testtiming:keep pins parameterized and fully qualified failure-name matching, another test's failure and an out-of-project compile error judging none, and the exact build-error text`."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirstdotnet.go:246
    summary: nearestProject drops the os.ReadDir error, so an unreadable directory reads as one with no .csproj and the search climbs to an ancestor project, grouping the test under the wrong project
    fix: "In nearestProject, when os.ReadDir returns an error that is not fs.ErrNotExist, return a reason naming the directory and the error, so the call exits through redFirstCannotJudge. Keep climbing past a directory that does not exist."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:502
    summary: The exit contract still says the envelope lists the red tests as `<package>.<Name>`, while a dotnet run lists them as `<Class>.<Method>` through dotnetLabels
    fix: "Reword doc.go:502 to say the envelope lists the red tests as `<package>.<Name>` under the go scoper and as `<Class>.<Method>` under the dotnet scoper."
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:999
    summary: The unjudged list names a hub whose scoper is neither go nor dotnet and C# tests no filter or tier can reach. It leaves out the case the code also exits unjudged on, a test file whose .csproj cannot be told.
    fix: "In docs/overview.md:999 extend the parenthesis to read: a hub whose scoper is neither `go` nor `dotnet`, C# tests no filter or tier can reach, or a C# test file whose project cannot be told."
---

### [BLOCKING] The base checkout leaves out every executable file

**Location:** `internal/gitrepo/blobread.go:226`

**Issue:** `WriteTreeAtRevision` filters with `file.Mode.IsRegular()`.
In go-git, `filemode.FileMode.IsRegular` returns true only for `Regular` and `Deprecated`; `IsFile` is the predicate that also admits `Executable`.
A blob stored as `100755` is therefore never written.
The `filemode.Executable` branch of `writeBlobFile` can never run, and both doc comments are false: the gitrepo one ("An executable file is written with mode 0o755") and fabric's `ExportCodeTreeAt` ("keeps its executable bit").
For red-first, a test or source file committed as executable is missing from the base checkout.
The base build then fails, or the missing file reads as a compile error and gives a false red.
The integration row commits no executable, so the test never sees this.

### [BLOCKING] New branches on the scoper literal "dotnet"

**Location:** `internal/gatecli/redfirst.go:135`

**Issue:** The diff adds `cfg.Scoper != "dotnet"` at 135 and 148 and `cfg.Scoper == "dotnet"` at 166.
PATTERN-code-conventions, "No hard-coded choice", says a language is a row in a table keyed by its name, and no code path branches on a language literal outside it.
It also says a change that touches an existing literal "never adds another".
`internal/gateslot/config.go` already keeps such a table, `scoperHasGoCoverageMaps`, behind `HasGoCoverageMaps`, and line 135 combines it with a new literal.
No commit carries a `Pattern-Departure:` trailer.

### [MEDIUM] A filter that selects no test reads as green

**Location:** `internal/gatecli/redfirstdotnet.go:282`

**Issue:** `judgeDotnetRedFirst` returns `redFirstGreen` for any exit 0.
When the `FullyQualifiedName~<Class>.<Method>` term matches nothing, VSTest only warns ("No test matches the given testcase filter", the warning `internal/gatecli/tierrun_spawn_test.go:64` plants), and the run exits 0.
Examples are a generic or parameterized fixture, whose real name carries its type or arguments, and the parser bounds the summary names.
The verb then exits `RedFirstGreenExit`, and the red-first gate holds the writer with "a test that passes before the fix does not show the bug".
No test ran, and the writer cannot fix the name the parser chose.
That goes against the entry's "halt only when … a reason the writer can act on" and its "judge red on the merge base".

### [MEDIUM] A failed restore or a build failure outside the test project fails the gate terminally

**Location:** `internal/gatecli/redfirstdotnet.go:136`

**Issue:** A run that fails without a named failing test and without a compile error under the test project is `redFirstUnjudged`.
Examples are a failed `dotnet restore`, or an MSBuild error from a head `.csproj` that references a project the base lacks.
`redFirstVerdictExit` then exits 1, and `loomshed` fails any exit other than the red-first codes terminally.
The summary's "What exits 1" states this on purpose.
The entry's fourth point asks the gate to halt only when the selection or build fails for a reason the writer can act on, and a transient restore failure is not one.
Nothing separates a run that never started its tests from one that ran them and judged none.

### [MEDIUM] Per-project grouping has no test

**Location:** `internal/gatecli/redfirstdotnet_test.go:90`

**Issue:** The entry selects tests "by test project", and `run` loops once per `byProject` group, adding to `red` and `unjudgedCode` as it goes.
Every row of the call test has one test project, so a mutant that merges the groups, or keeps only the last group's red, passes.

### [LOW] The checkoutRoot swap is pinned only by a test that skips without .NET

**Location:** `internal/gatecli/run.go:108`

**Issue:** The unit test stubs `runTiers`, so `runTest` never runs with `checkoutRoot` set.
`TestGateRedFirst_Dotnet` is the only test that reaches it, and it skips on a machine without `dotnet` or the NUnit packages.
On such a machine nothing checks that the shared build and the tier dirs resolve in the checkout.

### [LOW] Two mutation survivors are contracts a row could pin

**Location:** `internal/dotnetimpact/source_test.go#TestParseSource_TestMethods`

**Issue:** The survivor at `source.go:233` changes `statementEnd(i+2)`.
No expression-body row opens its body with a bracket, so skipping one more token goes unseen.
The survivor at `redfirstdotnet.go:327` drops `slices.Sort` in `filterTerms`, whose doc promises "sorted and without repeats".
Two changed overloads of one method give two equal terms, and `slices.Compact` removes only adjacent repeats, so the sort matters.
Every call-test row has a single test.
The survivors at `source.go:118` and `425` and `redfirstdotnet.go:212` are no finding: the first two clear state only a malformed member leaks, and the third is a count in an informational stderr line.

### [LOW] Three branches of run have no row

**Location:** `internal/gatecli/redfirstdotnet_test.go:90`

**Issue:** These have no test row:
- dropping abstract-class tests while the concrete ones still run, at `redfirstdotnet.go:72`;
- the exit-1 refusals when the base cannot be exported or the head files cannot be written;
- `keepHeadProjectFiles` removing a test-project file the working tree deleted.

### [LOW] TierFailureNames' non-tier-run rule has no row

**Location:** `internal/gateslot/failedtests_test.go:210`

**Issue:** The doc says a failure under any run other than a tier or guard run names none, which is the `underTier` condition.
The test's build block holds only lines excluded on other grounds, so dropping `underTier` would pass.

### [LOW] A contained test needs its keep marker

**Location:** `internal/gatecli/redfirstdotnet_test.go:23`

**Issue:** The redundancy report lists the judge test as contained in the call test.
Its assertions go further than the call test's:
- it matches parameterized and fully qualified failure names;
- it judges none for another test's failure and for a compile error outside the project;
- it checks the exact `buildError` text.

The call test asserts a bare-name red, a `CS0117` substring and the judged-none exit.
The test should stay, marked with `//testtiming:keep`.

### [LOW] nearestProject drops its directory read error

**Location:** `internal/gatecli/redfirstdotnet.go:246`

**Issue:** `entries, _ := os.ReadDir(...)` turns a permission or I/O error into "no .csproj here".
The loop then climbs to an ancestor `.csproj`, so a test can run under the wrong project, or a "no .csproj holds" reason can hide the real cause.

### [LOW] The exit contract names only the Go label form

**Location:** `internal/gatecli/doc.go:502`

**Issue:** The exit-0 bullet says the envelope lists the red tests as `<package>.<Name>`.
Under the dotnet scoper they are `<Class>.<Method>` through `dotnetLabels`.

### [NIT] The overview's unjudged list leaves out one case

**Location:** `docs/overview.md:999`

**Issue:** The code also exits unjudged when a test file's project cannot be told: no `.csproj` above it, or two in its nearest directory.
`internal/gatecli/doc.go` and the summary name that case; the overview does not.

## Rejected

- correctness lens finding 3, "a compile error in the test project makes every changed test red": this is the Go path's own rule.
  `judgeRedFirst` makes every changed test of a package red when the package failed outside any test.
  The entry asks for red judged "as for Go", and a project that does not build on the base cannot run any of its tests there.
  Making red only the tests in the erroring file would not close the gap the lens names: the feature test's red still passes the gate.
- affected-set lens finding 2, "the noteScoper text still says not go": the refusal now prints only for scopers that are neither go nor dotnet, and for them "the hub's scoper is %s, not go, so the step judged nothing" stays true.
- Focus check of the Go path: `redFirstNoTest` and `redFirstVerdictExit` keep the Go path's messages, envelope keys and exit codes, and an empty `terms` leaves `scopeInvocations` unchanged. No finding.
- Commit trailers: no commit in the range carries `Root-Cause:`, `Loosened-Check:` or `Pattern-Departure:`.
  The entry asks for a feature, not a bug fix, no existing test's limits were loosened, and no commit claims an agreement or an improvement. No finding.
- Merge danger: the `two-way` call holds.
  The diff changes no record or board format another binary reads, no config setting's meaning, deletes no data and pushes nothing outward, and the blast radius `gate` names what it can break.
- Stencil and doc wording: `darn-template-write.md` keeps every rule it had and adds only the C# behaviour. The recipe comment, `shedrecipe` and `loomshed` docs match the entry.
