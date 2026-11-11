---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/doc.go:225
    summary: The Tier plan table still gives scope diff as the template default of webster-fix, code-burler-fix and darn-write.
    fix: In the Tier plan table of internal/gateslot/doc.go, change the Scope cell of the Webster verify fixers row (webster-fix), the Code-Burler-Fix row (code-burler-fix) and the Darn-Write row (darn-write) from diff to coverage. Leave the PR-Review row at diff.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/coveragescope.go:230
    summary: coverageSelections and openSelector have no test, so D8's rule that a failed map read is a miss and runs the package whole is unpinned.
    fix: Add a table-driven tier0 test beside TestNarrowCoverage in internal/gatecli/coveragescope_test.go over coverageSelections. Rows are a request with empty coverageMaps, which must give a Miss with reason testcover.MissNoMap, and a second call for the same package, which must return the cached selection. Add a row for an unreadable maps directory when it is reachable without a hub.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/tiers_test.go:435
    summary: D6's criteria that the selected run's green entry carries its -run args, never serves as a base or a whole pass, and that the tier is counted once have no test.
    fix: Add a greenRecord row in the gatecli green tests where an entry recorded with Args of a selected run is skipped by newestWholeAncestor and does not satisfy covered for a request with empty args. Add a tierNumbers row where a plan holding a tier's own run and its selected run lists the tier once. The plan rows already pin the -run position in the run's own args.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/scope.go:281
    summary: No test drives computeScope with a coverage key, so D1's rules that scoper none and dotnet equal diff and that a Whole fallback is never narrowed are unpinned.
    fix: Add rows to TestGateTest_Scopes in internal/gatecli/scopes_integration_test.go, or a tier0 computeScope test, where a hub with a coverage key under scoper none computes the same scope as a diff key, and where a diff that falls back to whole runs whole under a coverage key. The existing explicit-targets row of TestWithGateScope covers un-narrowed targets.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/testcover/testselect.go:167
    summary: Selection.Recorded is set by Select with no assertion, so dropping it would make every test unrecorded and run whole with no test failing.
    fix: In the existing Select test of internal/testcover, assert that a usable map's Selection.Recorded equals the map's Tests and that a Selection with a Miss has an empty Recorded.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:780
    summary: The sample gates block shows webster-fix, darn-write and code-burler-fix at scope diff, beside the new Coverage scope section that says the template gives them coverage.
    fix: In the sample gates block of internal/gatecli/doc.go, set the three keys to scope coverage, or add a line saying the sample is a hand-written hub file that keeps diff.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/fabricengine/diff.go:118
    summary: The new CodeWorktreeChanges is outside the record's three packages and the change description does not name it, though it names the testcover addition.
    fix: Name fabricengine.CodeWorktreeChanges in the What changes list of _lym/landing/summary.md, with the reason that the selector reads working files through fabric. The method itself and its test assertion stay.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/coveragescope.go:141
    summary: A FindGuardTests error in an importing package is labelled as a Go file that does not parse, and the guard-scan refusal other packages give is lost.
    fix: In importerPackage, when the error from FindGuardTests satisfies impactset.IsGuardScanError, label the miss with the error's own text, or return it so computeScope refuses with guardScanRefusal as it does for a scan error elsewhere. Keep the whole run for any other read or parse error.
  - id: F9
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/impactset/targets_test.go:58
    summary: TestTargetsFor has no row where a package that is both an importer and builds the binary becomes a binary builder under a cmd/lym change, or stays an importer when cmd/lym is unchanged.
    fix: Add two rows to TestTargetsFor in internal/impactset/targets_test.go. In the first, cmd/lym and a package reached by imports whose tests build the binary change together, and the package is in BinaryBuilders. In the second, only that package's dependency changes, and the package is in Importers.
  - id: F10
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/coveragescope_test.go:116
    summary: TestNarrowCoverage and TestCoverageTier lack rows for an unresolved-marked test, an unparseable test file, a test with no tiergate.Require, tier.Covers tiers, an undeclared tier and a command with no target placeholder.
    fix: Add rows to TestNarrowCoverage for a test whose Spawn verdict is Unresolved, which joins the union, a test file that does not parse, which runs the package whole with a miss label, a test with no Require, which counts, and a tier whose Covers holds the other tier, which counts that tier's tests. Add rows to TestCoverageTier for an undeclared tier and a command with no target placeholder.
---

### [MEDIUM] Tier plan table in the gateslot doc names the old template scope

**Location:** `internal/gateslot/doc.go:225`
**Issue:** The diff updated the key format line and the scope list of this file, and left the table of shipped rows with scope `diff` for `webster-fix`, `code-burler-fix` and `darn-write`. `template.yaml` now gives `coverage`, so the table contradicts the template it documents.

### [MEDIUM] Selector setup has no test

**Location:** `internal/gatecli/coveragescope.go:230`
**Issue:** `TestNarrowCoverage` injects `selectionOf`, so `coverageSelections` and `openSelector` run in no test. D8's acceptance criterion that an unreadable map runs the package whole, and the outside-a-hub miss, rest on code no test reaches.

### [MEDIUM] The selected run's execution-side behaviors are unpinned

**Location:** `internal/gatecli/tiers_test.go:435`
**Issue:** `TestPlanTierRuns` pins the plan strings. The decision record's D6 criterion that the green entry carries the `-run` arguments and is never read as a pass or a base has no test, and neither has the `tierNumbers` dedupe. The change description admits the first gap. The code reads correct: the lens traced `newestWholeAncestor` and `covered`, which reject the entry. The tests that would break on a regression are missing.

### [MEDIUM] computeScope's coverage branch has no test

**Location:** `internal/gatecli/scope.go:281`
**Issue:** `TestGateTest_Scopes` was switched from `webster-fix` to `pr-review` to keep its diff meaning, so no test runs a coverage key through `computeScope`. D1's criteria that scopers `none` and `dotnet` equal `diff`, and that a whole fallback is not narrowed, are unpinned.

### [MEDIUM] Selection.Recorded is unasserted

**Location:** `internal/testcover/testselect.go:167`
**Issue:** The `Recorded` field is what lets the gate find tests the map does not record. Dropping it from `Select` would turn every test into an unrecorded one and run every importer whole, with every existing test still passing.

### [LOW] gatecli doc sample block still shows diff

**Location:** `internal/gatecli/doc.go:780`
**Issue:** The sample `gates:` block lists `scope: diff` for the three keys the template now defaults to `coverage`, in the file that gained a Coverage scope section.

### [LOW] fabricengine addition not in the change description

**Location:** `internal/fabricengine/diff.go:118`
**Issue:** The record scopes three packages and the description discloses the `testcover` addition, but not `CodeWorktreeChanges`. The method serves D8's rule that file contents come through fabric, and `PATTERN-fabric-boundary` holds, so the code is licensed in effect. The description does not bear it out.

### [LOW] Guard scan failure mislabelled as a parse miss

**Location:** `internal/gatecli/coveragescope.go:141`
**Issue:** `importerPackage` gives any error from `tierTestNames` or `FindGuardTests` the `Go file does not parse` reason. A misplaced `//lym:guard` marker is a `GuardScanError`, which `computeScope` refuses with its file, line and way forward for other packages. Here the package runs whole, which is safe, with a wrong reason in the scope report.

### [LOW] TestTargetsFor lacks importer and binary-builder boundary rows

**Location:** `internal/impactset/targets_test.go:58`
**Issue:** The rows cover a builder beside a `cmd/lym` change and a changed package that builds the binary. Neither the importer-that-also-builds-the-binary case nor the unchanged `cmd/lym` case has a row.

### [LOW] Narrow-coverage and coverage-tier rows are incomplete

**Location:** `internal/gatecli/coveragescope_test.go:116`
**Issue:** `marked[name]` is exercised for out-of-process only, and the unparseable-file miss, the no-`Require` rule, the `Covers` tiers, an undeclared tier and a command with no target placeholder have no row.

## Rejected

- Test economy: no new test duplicates an existing one, and the redundancy report judged none of the new tests, so no fold, delete or keep finding applies. The new tests are tier0 and parallel.
- The selected run's `-run` filter matching the guard-run argument shape when the call has no words after `--`: `argsMatch` then lets an earlier empty-args entry for the same tree cover the selected run. Such an entry ran every test of its packages, so the cover is correct.
- Scope of `testcover.Selection.Recorded`, `loomcli/landingdeps_verify_test.go` and `pattern/PATTERN-test-speed.md`: the first is disclosed and needed by D3, the second follows from the template default change, and the third is the repository's doc rule for the key table.
- `ScopeCoverage` exported beside unexported `scopeDiff` and `scopeAll`: gatecli needs the value across packages, and the other two are used only inside gateslot.
- No `Pattern-Departure:` trailer in any commit message, and no declared PATTERN change in the directive, so nothing to check there.
