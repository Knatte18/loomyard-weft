---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/redfirst.go:154
    summary: The No test seam title and the bug label are repeated literals, breaking PATTERN-code-conventions' one-named-constant rule
    fix: "Declare the title once as an exported constant in internal/gateslot (beside RedFirstCall), and the bug label once beside it. Format the gatecli no-test refusal at internal/gatecli/redfirst.go:154 from the title constant with %q instead of the literal \"No test seam\". In internal/loomshed/redfirstgate.go, delete the local NoTestSeamTitle and BugLabel declarations and reference the gateslot constants in NewRedFirstGate, redFirstFailure and the loomshed doc.go paragraph. Leave the older labelBug, fixLabelBug and defaultLabel literals in reportengine, adoptcli and selfreportengine as they are; this finding asks only that the diff add no further declaration."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/gatecli/redfirst.go:119
    summary: "On a hub whose scoper is not go, lym gate red-first exits 0, the code for a red test, so the Darn-Write gate silently passes every bug run there"
    fix: "In runRedFirst, replace the output.Ok note return for !cfg.HasGoCoverageMaps() with a new exit code gateslot.RedFirstUnjudgedExit (declared beside RedFirstGreenExit in internal/gateslot/redfirst.go) and an error envelope naming the scoper. Its way forward: the run's parent records an added decision under the No test seam title. In internal/loomshed/redfirstgate.go redFirstFailure, add a non-terminal case for that exit code whose findings say the hub's scoper cannot run the red-first check and give the same creel ask to the parent that the RedFirstNoTestExit case gives. Update the gatecli, gateslot and loomshed doc.go text, the verb's Long help, docs/overview.md and the summary to match. Add a row to TestRunRedFirst_RefusesAMissingGateKeyWithAWayForward, or a sibling table row, that runs the verb on a hub with scoper none and asserts the new exit code. Also add a row to TestRedFirstGate_VerdictPerLabelRecordAndExitCode for that exit code."
    origin: lens:code-scope
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/redfirst.go:266
    summary: judgeRedFirst drops a changed test's named failure whenever the failure report also holds a cut, cause or unparsed line, because FailedTests returns nil for any such report
    fix: "Add a gateslot reader beside PackagesFailedOutsideTests, for example NamedFailedTests(log) []FailedTest. It returns every `  <package> <Test>:` line of the first failure report, each once, up to the cut line, ignoring other lines rather than giving up. Use it in judgeRedFirst in place of gateslot.FailedTests. Keep FailedTests unchanged for its rerun callers. Add rows to TestJudgeRedFirst_ReadsTheRunsExitCodeAndFailureReport for a report that names TestClamp beside a ReportCutPrefix line, and one that names it beside another package's failed-outside-any-test line; both want redFirstRed with clamp. Add a table test for the new reader in internal/gateslot/failedtests_test.go. Drop the bound about a mixed report from internal/gatecli/doc.go and the summary."
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gatecli/redfirst.go:327
    summary: "The base run keeps HEAD's changed non-Go files of the module (embedded stencils, recipes, testdata) and HEAD's prebuilt lym, so a correct fix there is judged green and the writer is sent after a test that cannot exist"
    fix: "In baseOverlay, also collect each changed path inside the module's packages that is neither a .go file nor under a testdata directory. When that set is non-empty, return it to redFirstCall.run. run then exits with the unjudged exit code from F2 before running any tier, with an error naming those paths. The message says the overlay cannot put them back and the parent records the No test seam decision. Add a row to TestBaseOverlay_ReplacesOnlyTheProductionGoFilesOfTheModule, or a new return value check, for a changed sub/a/data.json. In internal/gatecli/doc.go, replace the bounds about non-Go files and the prebuilt lym binary with this refusal. Keep the prebuilt-binary bound only as a stated limit, since no overlay reaches it."
    origin: assist:assist
  - id: F5
    severity: MEDIUM
    class: decision
    location: contracts/recipes/darn-recipe.yaml:140
    summary: "The board entry asks for the gate on any entry labelled bug, but only Darn-Write carries it; tapestry and sampler runs of a bug entry are not checked, and no Added after Discussion entry records that narrowing"
    fix: "Leave the code in place and defer this finding to the run's parent. Its only sound change is recording or refusing the narrowing to darn with lym loom decision add, which lies outside the fixer's write surface. If the parent refuses the narrowing, a later round adds the red-first gate to the tapestry and sampler write rows."
    origin: lens:code-scope
  - id: F6
    severity: MEDIUM
    class: consistency
    location: cmd/lym/verifiedtree_test.go:19
    summary: The new verifytree.Verify site is missing from the verify-site tripwire, from PATTERN-gate-slot's Verified tree section, and from Verify's doc, which still says its command is only lym gate test --wait calls
    fix: "Add internal/loomshed/redfirstgate.go to planVerifySites in cmd/lym/verifiedtree_test.go. In pattern/PATTERN-gate-slot.md's Verified tree section, add the Darn-Write red-first gate to the site list. Say there that its command is the lym gate red-first call gateslot.RedFirstCall renders for the darn-write key, which takes its slots through the in-process tier run. In the Verify doc comment in internal/verifytree/verify.go, widen the sentence about the command to name a lym gate red-first call beside lym gate test --wait calls."
    origin: lens:code-affected-set
  - id: F7
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/loom/loom-rubric-code-review.md:90
    summary: The rubric's missing-trailer clause applies to every recipe's Code-Review, but only the darn writer is asked for a Root-Cause trailer, so a tapestry or sampler bug run draws a finding its writers were never asked to avoid
    fix: "Reword the bullet's last sentence so the missing-trailer finding applies only to a darn run, the one whose writer is asked for the trailer. For example: 'When the answer key asks to fix a bug in a darn run and no commit of the range carries the trailer, that is a finding too.' Leave the first two sentences as they are."
    origin: lens:code-affected-set
  - id: F8
    severity: MEDIUM
    class: consistency
    location: "_lym/landing/summary.md#Merge danger"
    summary: "The blast radius names only darn, but the diff changes the Code-Review rubric that the tapestry and sampler recipes' Code-Review perches also render"
    fix: "In _lym/landing/summary.md's Merge danger section, widen the blast radius to name darn and the Code-Review perch of every loom recipe (tapestry, sampler and darn), whose shared rubric gains the Root cause of a bug fix bullet."
    origin: handler
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/loomcli/wiring.go:593
    summary: "entryLabels, which decides whether the red-first gate runs at all, is reached only by wire tests that check the seam is non-nil; its board read, its absent-entry case and its error wrap are untested"
    fix: "Add a test in internal/loomcli beside the wiring tests that builds a hubforge hub with a board entry whose slug is the worktree name and whose labels include bug. Call entryLabels(location)() and assert it returns the entry's labels exactly. Relabel the entry and call the same closure again, asserting the new labels, which pins the per-call board read. Add a row for a worktree with no board entry that asserts nil labels and no error."
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: design
    location: internal/gatecli/redfirst_test.go:676
    summary: "The mutation survivor at redfirst.go:287 (negated condition in changedTestsAt) passes TestChangedTestsAt, though commit 8399cbd0 says the test pins which packages are compared"
    fix: "In TestChangedTestsAt_ComparesTheTestFilesOfEachPackageHoldingAChangedOne, add a package sub/c whose only changed path is sub/c/c_test.go. It has a test that differs between base and head, with no production file of sub/c in changed. Add its changed test to want. The negated condition then drops sub/c and the test fails."
    origin: lens:code-tests
  - id: F11
    severity: LOW
    class: design
    location: internal/loomshed/redfirstgate_test.go:174
    summary: "TestRedFirstGate_VerdictPerLabelRecordAndExitCode has no row for the case-insensitive No test seam match, a decision-record read or parse error, or the timed-out and runner-crash terminal branches"
    fix: "Add rows to the table: a record whose added entry title opens 'no test seam' in lower case (want passed, not ran); a bug-labelled entry whose recordPath does not exist (want a returned error, not ran). Add a command whose log holds the gate's tier timeout line and exits 1 (want terminal, findings containing 'timed out'), building that line from the verifytree constant the verify gate tests already use."
    origin: lens:code-tests
  - id: F12
    severity: LOW
    class: design
    location: internal/shedrecipe/entries_gate_test.go:63
    summary: No test asserts that the red-first entry renders env.RedFirstCommand for the row's own gate key
    fix: "Add a test beside TestResolveGateSpec_RedFirstRequiresItsEnv that sets env.RedFirstCommand to a closure recording its argument. It resolves gatesCfg(\"red-first\", 3) for row Darn-Write and asserts the recorded key is the gate key that row's verify entry receives (the same key the existing VerifyCommand row-key cases assert)."
    origin: lens:code-tests
  - id: F13
    severity: LOW
    class: design
    location: internal/gatecli/redfirst.go:210
    summary: The -run pattern is the union of every changed test's name across all target packages, so an unchanged test sharing a changed test's name in another target package also runs, and its failure on the base can turn a green run into an unjudged terminal halt
    fix: "In redFirstCall.run, when the changed tests span more than one package and a name in one package also names a test of another target package, run the tiers once per package with that package's names only. Otherwise keep the single run. Simplest form: group c.tests by Package and call runTest once per group, merging the captured output before judgeRedFirst. Add a TestJudgeRedFirst row, or an integration row, with two packages sharing a test name."
    origin: assist:assist
---

### [BLOCKING] The No test seam title and the bug label are repeated literals

**Location:** internal/gatecli/redfirst.go:154

**Issue:** `PATTERN-code-conventions` requires a name that can go stale to be "one named constant, declared once and referenced everywhere, never a repeated literal".
The diff declares `loomshed.NoTestSeamTitle`, yet the gatecli no-test refusal spells `"No test seam"` again as a literal, so a rename of the title leaves the CLI's way forward asking for an entry the gate no longer matches.
`loomshed.BugLabel` adds one more declaration of the `bug` label beside `reportengine`'s `labelBug`, `adoptcli`'s `fixLabelBug` and `selfreportengine`'s `defaultLabel`.
The round's constraint rule makes every PATTERN violation BLOCKING.

### [BLOCKING] A non-Go hub silently passes every bug run

**Location:** internal/gatecli/redfirst.go:119

**Issue:** `runRedFirst` returns `output.Ok` with exit 0 when `cfg.HasGoCoverageMaps()` is false.
Exit 0 is the verb's "a changed test went red" code, and `NewRedFirstGate` treats it as `StatusPassed` (`internal/loomshed/redfirstgate.go:104`), recording a verified pass for the tree.
Every bug run on a `none` or `dotnet` hub therefore hands off with no test judged and no record naming a stop: the "silent pass" the board entry's goal forbids.
The non-Go path also has no test.

### [MEDIUM] Named red failures are lost in a mixed or cut failure report

**Location:** internal/gatecli/redfirst.go:266

**Issue:** `gateslot.FailedTests` returns `nil, false` as soon as the report holds a line that is not a named test of a tier run: a cut line past the report's line cap, a cause line, unparsed output or another package's build failure (`internal/gateslot/runlog.go:296–330`).
`judgeRedFirst` discards `ok` and reads the nil list.
A changed test that did fail by name on the base is then judged "none", and the gate fails terminally (`internal/loomshed/redfirstgate.go:137`), halting a correct run for the parent.

### [MEDIUM] The base run keeps HEAD's non-Go inputs

**Location:** internal/gatecli/redfirst.go:327

**Issue:** `baseOverlay` restores only changed `.go` files.
In this repository many fixes land in embedded files: stencils, recipes and specs under `contracts/`, read through `//go:embed`.
For such a fix the "base" run still embeds HEAD's fixed file, so the fix's test passes there, the verb exits `RedFirstGreenExit`, and the writer is told to "make a test reach the bug through code the base already has".
No such test can exist.
The run burns its three attempts and stops, and the only way out is a No test seam record for a bug that has a seam.
The summary's Bounds list this, but a known false green should be refused as unjudged, not reported as green.

### [MEDIUM] The gate covers only darn runs

**Location:** contracts/recipes/darn-recipe.yaml:140

**Issue:** The amended board entry reads "For an entry labelled `bug`, a gate step runs …" with no recipe qualifier.
Only the darn recipe's Darn-Write row lists `red-first`, and the change description states that tapestry and sampler runs of a bug entry are not checked.
No `Added after Discussion` entry records this narrowing, so the parent must record or refuse it.

### [MEDIUM] Verify-site documents and tripwire miss the new site

**Location:** cmd/lym/verifiedtree_test.go:19

**Issue:** `internal/loomshed/redfirstgate.go` calls `verifytree.Verify` with a `Hold`, but `planVerifySites` does not list it, so nothing keeps it naming a hold.
`pattern/PATTERN-gate-slot.md`'s Verified tree section lists the verify sites and says their commands are `lym gate test --wait` calls; it lacks this site and its command.
`Verify`'s own doc comment says the same.

### [MEDIUM] The rubric's missing-trailer clause reaches recipes that never ask for the trailer

**Location:** contracts/stencils/loom/loom-rubric-code-review.md:90

**Issue:** `loom-rubric-code-review` is the rubric stencil of the Code-Review perch in the tapestry, sampler and darn recipes.
Only `darn-template-write.md` asks the writer for `Root-Cause:`.
A tapestry or sampler run fixing a bug therefore gets a finding for a trailer its writers were never told to write.

### [MEDIUM] The merge-danger blast radius names less than the diff touches

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The blast radius reads `darn`.
The diff also changes the Code-Review rubric shared by every loom recipe's Code-Review perch, so tapestry and sampler reviews change behaviour too.

### [MEDIUM] entryLabels has no behaviour test

**Location:** internal/loomcli/wiring.go:593

**Issue:** The closure opens the board and reads the run's entry on each call, and its result alone decides whether the red-first gate runs.
The reach report shows it reached only by `TestWire_*` tests, which assert the seam is set.
A closure that read another slug, cached the first answer or always returned no label would pass every test and silently disable the gate.

### [LOW] changedTestsAt's package selection is not pinned

**Location:** internal/gatecli/redfirst_test.go:676

**Issue:** The mutation report's survivor at `redfirst.go:287` negates the `_test.go` and module-package condition, and the test still passes.
The fixture's only changed test package, `sub/a`, also holds a changed production file, so the negated condition still selects it.

### [LOW] The gate's record, case and terminal branches lack rows

**Location:** internal/loomshed/redfirstgate_test.go:174

**Issue:** The fixture's added entry title uses the exact case, so dropping `strings.ToLower` survives.
No row reaches the record read error, the parse error or the `TimedOut` branch of `redFirstFailure`.

### [LOW] The red-first command's gate key is not asserted

**Location:** internal/shedrecipe/entries_gate_test.go:63

**Issue:** The vocabulary test only checks that the `red-first` entry resolves.
Passing `""` or another row's key to `env.RedFirstCommand` at `entries_gate.go:335` passes every test.

### [LOW] One -run union runs same-named unchanged tests in other target packages

**Location:** internal/gatecli/redfirst.go:210

**Issue:** For changed `a.TestA` and `b.TestB`, both `./a` and `./b` run `-run=^(TestA|TestB)$`.
An unchanged `a.TestB` runs too.
`judgeRedFirst` filters by package, so it cannot wrongly turn a run red.
But if that unrelated test fails on the base while every changed test passes, the run exits non-zero with no changed test red, which the gate reads as unjudged and halts terminally.

## Rejected

- Assistant F1, that a package failing outside any test (a compile failure, a TestMain exit) must not count as red: the board entry says the gate fails "when every one passes there", and a test that does not build on the base does not pass. Counting a compile failure as red is the usual TDD reading, and the summary states it as a design decision. The writer stencil already asks for a test that reaches the bug through code the base already has.
- Assistant F3, that a No test seam record must stop the run rather than let the gate pass: the stop happens when the writer ends its turn and waits for the parent. The parent's recorded entry is the record that names the stop. The gate then logs the entry's title rather than passing silently, which meets "a stop the record names".
- Assistant F12, the `runlog.go:424` survivor: changing the `start+1` offset either re-reads the header, which the two-space prefix check skips, or skips the run-label line, which never names a package. It is an equivalent mutant.
- Code-tests finding 3, integration rows for a test that fails to build on the base and for an added production file: `judgeRedFirst`'s outside-any-test row and `TestBaseOverlay`'s base-lacks row pin the verb's own logic. That `go test -json` reports a build failure as failed outside any test, and that an empty overlay target deletes a file, are existing contracts of the gate's counter and of Go.
- Mutation survivors at `redfirst.go:87` and `:88` (the flag definitions): `TestGateRedFirst` passes `--gate` and `--base` and fails without them. The mutation run did not see that tier-1 test.
- Mutation survivor at `redfirst.go:353` (the overlay file's permission constant): no test can reasonably observe it.

## Focus verdicts

- Does the gate fail only when every changed test passes on the base, and how does it count a build failure? Exit 0 from the tier run is green, and a named failure or a failed-outside-any-test package is red. A build failure is red, which I accept (see Rejected). The defects are F2 (a non-Go hub reports red) and F3 (a mixed report loses a named failure).
- Does the overlay replace only production Go files, and does it handle added, deleted and renamed files? Yes. `ChangedPaths` lists both sides of a rename: the base-held side maps to a copy, and a file the base lacks maps to `""`. Non-Go inputs are F4.
- Is the gate applied only to a `bug` entry, and is a missing seam a named stop? Yes on both, in darn only (F5).
- Does the trailer reach Code-Review, and do the stencil edits drop or weaken a rule? The trailer reaches Code-Review, and no rule is dropped or weakened. The missing-trailer clause over-reaches into recipes whose writers are never asked for it (F7).
- Test economy and the survivors: F9 through F12. The `runlog.go:424` and `:353` survivors are no findings.
- Docs and claims: the doc and overview text match the code. The summary's blast radius is F8.
