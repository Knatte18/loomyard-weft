---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "internal/gatecli/redfirst.go:270"
    summary: "Package failures certify tests that never ran"
    fix: "Require a named changed-test failure for red-first success. Keep build, TestMain, startup and infrastructure failures unjudged; correct the existing outside-test verdict row and add a setup-failure regression case."
    origin: "lens:code-correctness"
  - id: F2
    severity: BLOCKING
    class: scope
    location: "internal/gatecli/redfirst.go:120"
    summary: "Unsupported scopers pass the bug gate without testing"
    fix: "Replace the successful unsupported-scoper return with a nonzero actionable unjudged outcome that stops the gate. Implement a configured-language base-test route before allowing that scoper to produce a successful red verdict."
    origin: "lens:code-scope"
  - id: F3
    severity: BLOCKING
    class: scope
    location: "internal/loomshed/redfirstgate.go:63"
    summary: "A recorded no-seam reason automatically advances the run"
    fix: "Return a terminal failing gate result naming the matching no-seam decision instead of Passed true. Update the existing exemption test and the recipe, writer and package wording to describe the required recorded stop; a reason alone must not authorize a waiver."
    origin: "lens:code-scope"
  - id: F4
    severity: BLOCKING
    class: scope
    location: "contracts/recipes/darn-recipe.yaml:140"
    summary: "Bug runs outside darn never reach the red-first gate"
    fix: "Wire the label-conditioned red-first check into the completed-code paths of tapestry and sampler as well as darn, using the recipe registry and each site's configured gate key. Give the corresponding writers the root-cause trailer instruction and retain the non-bug fast path."
    origin: "lens:code-scope"
  - id: F5
    severity: BLOCKING
    class: design
    location: "internal/gatecli/redfirst.go:210"
    summary: "The base execution retains HEAD production inputs and child binaries"
    fix: "Create a base production execution view that retains the changed tests and required test support while restoring production resources and dependency inputs. Build subprocess binaries from that same view, including the shared lym prebuild and test-triggered builds. Extend the existing integration behavior with an embedded-resource fix and a subprocess-visible fix; report unsupported execution views as unjudged rather than all-green."
    origin: "lens:code-correctness"
  - id: F6
    severity: BLOCKING
    class: design
    location: "internal/gatecli/redfirst.go:154"
    summary: "The recovery message duplicates the no-seam title discriminator"
    fix: "Move the no-seam title discriminator to a shared package usable by the gate and CLI, and format the CLI recovery text from that constant. Reference the same constant in the gate matcher and its generated recovery messages."
    origin: "lens:code-affected-set"
  - id: F7
    severity: MEDIUM
    class: design
    location: "internal/gatecli/redfirst.go:266"
    summary: "The verdict reader discards observed failures in mixed or truncated reports"
    fix: "Collect named changed-test failures from structured tier results or a report reader that retains them independently of rerun eligibility. Keep the existing strict FailedTests contract for its other callers, and extend the verdict test with mixed and truncated reports containing a named changed-test failure."
    origin: "lens:code-correctness"
  - id: F8
    severity: MEDIUM
    class: design
    location: "internal/gatecli/redfirst.go:209"
    summary: "A global name filter runs unchanged tests in other selected packages"
    fix: "Build package-specific test-name filters and run each selected package with its own changed-test names. Extend TestGateRedFirst with changed tests in multiple packages and an observable unchanged test whose name matches a changed test in another package; assert that it never runs."
    origin: "handler"
  - id: F9
    severity: MEDIUM
    class: design
    location: "internal/loomcli/wiring.go:593"
    summary: "The real board-label lookup has no behavior test"
    fix: "Extend loom wiring behavior coverage to invoke EntryLabels for the run's task, distinguish another task's labels, change the run task's labels and invoke it again. Assert the new labels drive the next red-first gate arrival."
    origin: "lens:code-tests"
  - id: F10
    severity: MEDIUM
    class: design
    location: "internal/loomshed/redfirstgate.go:54"
    summary: "The gate adapter's record and verification failure paths lack coverage"
    fix: "Extend TestRedFirstGate_VerdictPerLabelRecordAndExitCode with missing and malformed records, dirty and already-verified trees, and timeout and runner-crash results. Assert returned errors, pass/terminal status, diagnostic text and absence of command execution where required; use controlled fixtures rather than production waits."
    origin: "lens:code-tests"
  - id: F11
    severity: MEDIUM
    class: design
    location: "internal/shedrecipe/entries_gate.go:335"
    summary: "Red-first command routing has no gate-key assertion"
    fix: "Extend the existing row-key cases in TestResolveGateSpec_ValidOneEntryPerVocabularyName to cover RedFirstCommand with both the derived row key and an explicit gate_key override. Assert the exact key received by the renderer."
    origin: "lens:code-tests"
  - id: F12
    severity: LOW
    class: design
    location: "internal/gateslot/runlog.go:424"
    summary: "Normal report boundaries have no exclusion assertion"
    fix: "Add rows to TestPackagesFailedOutsideTests with a valid package-failure-shaped line after a blank separator and after the report's closing line. Assert that neither later line is returned."
    origin: "lens:code-tests"
---

### [BLOCKING] Package failures certify tests that never ran

**Location:** internal/gatecli/redfirst.go:270

**Issue:** A package failure is not a failed regression test.
The counter records any unexplained package failure in `observePackageOutcome` (`internal/gatecli/counters.go:331`).
The report renders it as failed outside any test (`internal/gatecli/failurereport.go:280`).
`judgeRedFirst` then marks every changed test in that package red.
A `TestMain` that exits before `m.Run` therefore satisfies the gate without executing a changed test.
A compilation failure also supplies no evidence that the regression assertion exposed the bug.

### [BLOCKING] Unsupported scopers pass the bug gate without testing

**Location:** internal/gatecli/redfirst.go:120

**Issue:** `HasGoCoverageMaps` is false for `none` and `dotnet` (`internal/gateslot/config.go:76`).
The new verb returns success before selecting a test or validating the requested key.
`NewRedFirstGate` accepts that success at `internal/loomshed/redfirstgate.go:75`.
The board's bug-label requirement has no language exemption, and no no-seam reason is recorded on this path.
This consolidates scope finding 2 and correctness C4.

### [BLOCKING] A recorded no-seam reason automatically advances the run

**Location:** internal/loomshed/redfirstgate.go:63

**Issue:** The goal requires a bug without a test seam to stop with the reason recorded.
The prefix match instead returns `Passed: true` immediately after logging the title.
The title alone neither requires an explicit waiver nor proves any test red.
The existing test deliberately expects this advance, so its expectation must change.

### [BLOCKING] Bug runs outside darn never reach the red-first gate

**Location:** contracts/recipes/darn-recipe.yaml:140

**Issue:** Only Darn-Write lists the new gate.
Tapestry's implementer proceeds through Parent-Align and Drift-Verify toward Code-Review without it.
Sampler's session proceeds through Drift-Verify without it.
The relevant paths are `contracts/recipes/tapestry-recipe.yaml:338` and `contracts/recipes/sampler-recipe.yaml:104`.
The decision record contains no amendment narrowing the goal to darn.
The landing summary acknowledges the omission but cannot authorize it.

### [BLOCKING] The base execution retains HEAD production inputs and child binaries

**Location:** internal/gatecli/redfirst.go:210

**Issue:** The overlay restores only changed production Go files.
A fix to an embedded JSON mapping remains at HEAD during the supposed base run (`baseOverlay`, line 328).
Its regression test therefore sees the fixed mapping and is rejected as green.
The overlay also exists only in the outer test command's arguments.
`runInvocations` calls `prebuildLym` without it (`internal/gatecli/tierrun.go:377`).
That build uses the working tree (`internal/gatecli/prebuild.go:66`).
The fallback `lymbin.compile` likewise builds from the working tree (`internal/testkit/lymbin/lymbin.go:63`).
A CLI regression test consequently exercises the fixed binary.
These are one execution-view defect, consolidating correctness C3/C5 and scope finding 4.
Documenting the limitations does not satisfy the requested merge-base run.

### [BLOCKING] The recovery message duplicates the no-seam title discriminator

**Location:** internal/gatecli/redfirst.go:154

**Issue:** `NoTestSeamTitle` defines the record contract in `internal/loomshed/redfirstgate.go:25`.
The CLI independently spells the same title in its production refusal.
A change to the discriminator can leave the recovery instruction requesting an entry the matcher no longer recognizes.
`pattern/PATTERN-code-conventions.md:11` requires one named constant for such a name.
The round's constraint rule makes this violation BLOCKING.

### [MEDIUM] The verdict reader discards observed failures in mixed or truncated reports

**Location:** internal/gatecli/redfirst.go:266

**Issue:** `FailedTests` is an all-or-nothing rerun-eligibility parser.
Its branches return `nil, false` on a package failure, cause line or cut report (`internal/gateslot/runlog.go:301`).
Ignoring the Boolean does not preserve the named failures it already saw.
The report renderer caps its output (`internal/gatecli/failurereport.go:197`).
A regression test with enough failing subtests can thus be visibly red yet produce the terminal unjudged result.
A mixed report with an unrelated package failure loses the same evidence.

### [MEDIUM] A global name filter runs unchanged tests in other selected packages

**Location:** internal/gatecli/redfirst.go:209

**Issue:** The command combines every selected package with the union of every selected test name.
For changed `a.TestA` and `b.TestB`, both packages receive `-run=^(TestA|TestB)$`.
An unchanged `a.TestB` therefore runs too, contrary to the changed-tests-only requirement.
Its failure can contaminate the report or consume the tier's time budget.
The integration test supplies one changed test in one package and cannot detect this cross-product.
This incorporates the execution-selection coverage gap from code-tests finding 2.

### [MEDIUM] The real board-label lookup has no behavior test

**Location:** internal/loomcli/wiring.go:593

**Issue:** The new callback opens the board and resolves the run's task on each call.
The seam-completeness test checks only that it is non-nil (`internal/loomcli/wiring_seams_test.go:178`).
The gate test supplies fixed callbacks (`internal/loomshed/redfirstgate_test.go:23`).
Neither catches a callback that returns no labels, reads another task or caches the initial labels.
Those errors disable or misapply the mandatory bug gate.

### [MEDIUM] The gate adapter's record and verification failure paths lack coverage

**Location:** internal/loomshed/redfirstgate.go:54

**Issue:** The new adapter adds record errors, dirty and cached-pass handling, and distinct terminal timeout/crash findings.
Its behavior table covers ordinary exit codes and a label-reader error only.
The underlying verification tests do not check this adapter's translation.
A wrong pass or nonterminal result on these paths would therefore evade the new tests.

### [MEDIUM] Red-first command routing has no gate-key assertion

**Location:** internal/shedrecipe/entries_gate.go:335

**Issue:** The red-first constructor passes the resolved row key to the new command renderer.
The vocabulary case checks closure presence, name and attempts only.
The adjacent key-routing cases exercise `VerifyCommand` alone (`internal/shedrecipe/entries_gate_test.go:112`).
An empty or unrelated key supplied specifically to `RedFirstCommand` remains undetected.

### [LOW] Normal report boundaries have no exclusion assertion

**Location:** internal/gateslot/runlog.go:424

**Issue:** The parser promises to read the first report only.
Its test covers a cut marker but places no package-shaped line after either ordinary boundary.
The reported mutation survivor is consistent with this missing assertion.
The proposed rows pin the parser's public contract, rather than its loop structure.

## Review notes

Reviewed commits from `7346d6a8c8b38d0b26987969ac76b6b3843727c6` through HEAD.
The four lens reports are saved beside this report.
The root-cause rubric addition reaches the shared Code-Review stencil and preserves the Loosened-Check rule.
Added, removed and renamed production Go paths are accounted for by the changed-path list and overlay mapping.
The changed tests add no fixed sleeps or production-duration waits.
No grounded redundant-test finding was identified.
The changedTestsAt condition survivor does not establish an observable defect by itself:
its output is also filtered by testreach's module rules.
The overlay file-mode survivor does not establish a regression in the red-first behavior.

## Validation

A focused tier0 gate invocation was attempted for gatecli, gateslot and testreach.
No tests ran: the sandbox refused writes to the hub's gate-slot records under `_bolt/.lym/gate`.
This is a validation limitation, not a finding against the diff.

