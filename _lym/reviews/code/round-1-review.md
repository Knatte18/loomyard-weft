---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/verifygate.go:462
    summary: "outputAboveHeader searches the whole log for the first matching FAIL header and ignores the failing package, so a test name shared by two packages is given another package's output."
    fix: "In internal/loomshed/verifygate.go, scope outputAboveHeader to the failing package's block. Pass failure.Package from failuresSection and search only the lines after the previous package-closing line (an `ok  \\t`, `FAIL\\t` or `?   \\t` line) up to the package's own `FAIL\\t<package>` line, as parseVerifyFailures does with blockStart. Also stop the upward scan at those package lines, so the closing lines of a previous package never become a test's output. Add a TestVerifyFailureFindings row where two packages fail a test of the same name and each failure gets its own output."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:426
    summary: "The retry wiring has no covering test: removing the retryOnShortage call in runGreenAware, the shortage propagation in report or the failedRunLabel call in runTest fails no test, so the retry-passes and twice-failed acceptance criteria are pinned only on fakes."
    fix: "Add one row to TestGateTest_Scenario in internal/gatecli/test_integration_test.go. The stand-in prints `Resource temporarily unavailable` and exits non-zero on its first run, then passes on its second, with shortageRetryDelay set to 0 in the request. Assert exit 0 and the retry line on stderr. A second row where the stand-in fails with the marker both times asserts the original exit code and a closing `gate test: failed runs:` line that names a resource shortage."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/verifygate.go:437
    summary: "failuresSection quotes the first failure whole whatever its size, so one very long output line exceeds the verifyFailuresBytes bound without limit; the byte bound and its More failures line have no test."
    fix: "In failuresSection, truncate an entry longer than verifyFailuresBytes on a rune boundary (utf8.RuneStart) with a truncation marker before the `b.Len() > 0` check. Update the verifyFailuresBytes comment to match. Add a TestVerifyFailureFindings row with enough failing tests to pass the bound and one oversized output, asserting the first failure is present, the later ones are cut, and the More failures line appears."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/test_integration_test.go:792
    summary: "Acceptance criteria are asserted only partly: the build step's build-server variables, a tier's own env overriding them, and a set dir such as sub staying root-relative under a subdirectory anchor have no assertion."
    fix: "In internal/gatecli/test_integration_test.go, record MSBUILDDISABLENODEREUSE and DOTNET_CLI_USE_MSBUILD_SERVER in the stand-in's build mode and assert them in the existing build scenario. Add a tier whose env sets MSBUILDDISABLENODEREUSE and assert its value wins. Extend TestGateTest_UnsetDirFollowsTheAnchor with a tier whose dir is a subdirectory and assert it runs in that subdirectory under the worktree root."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun.go:226
    summary: "The branch that returns the first result when the call's context is cancelled during the retry delay is untested; only a context cancelled before the call is covered."
    fix: "Add a TestRetryOnShortage case in internal/gatecli/tierrun_test.go with a positive delay and a context cancelled by the first attempt, asserting one attempt ran, the first result was returned and nothing was announced as rerun."
---

### [MEDIUM] Failure output is taken from the first same-named header in the whole log

**Location:** internal/loomshed/verifygate.go:462
**Issue:** `failuresSection` derives the test name with `TrimPrefix(ID, Package+".")` and `outputAboveHeader` then scans the entire log for the first `--- FAIL: <test> ` line, so two packages that fail a test of the same name get the earlier package's output, or none. The re-prompted agent reads another package's evidence as this failure's. The same unscoped upward scan also pulls the previous package's closing lines into a test's output.

### [MEDIUM] The retry wiring is covered only on fakes

**Location:** internal/gatecli/run.go:426
**Issue:** `TestRetryOnShortage` drives `retryOnShortage` over fake attempts and `TestFailedRunLabel` a hand-built `runEnd`. Nothing drives `runTest`, so the acceptance criteria that a retried tier ends passed with the retry line on stderr and that a twice-failed tier is named a resource shortage in the closing line are not pinned where they happen.

### [LOW] The first failure's size is unbounded

**Location:** internal/loomshed/verifygate.go:437
**Issue:** The 40-line caps bound lines, not bytes, so one long line makes the first entry exceed `verifyFailuresBytes`. The bound and its overflow line have no test row.

### [LOW] Acceptance criteria asserted only in part

**Location:** internal/gatecli/test_integration_test.go:792
**Issue:** Only the tier run's environment is asserted. The build run, a tier `env` override and a set subdirectory `dir` under a subdirectory anchor are stated in the decision record and docs but pinned by no test.

### [LOW] Cancellation during the retry delay is untested

**Location:** internal/gatecli/tierrun.go:226
**Issue:** The `ctx.Done()` arm of the select has no case; the cancelled-before-start row returns earlier.

## Rejected

- Code-correctness F3 as its own finding (upward scan not stopping at package lines): merged into F1, whose fix scopes the scan to the package block.
- Code-affected-set reported no findings; the handler's own check of every `Tier.Dir` and `Build.Dir` reader agrees, since all read the config `WithDefaultDir` fills.
- Test-economy redundancy: none found; the new tests each cover a behavior no existing test covers.
- No `Pattern-Departure:` trailer is present or needed; the decision record declares no PATTERN change.
