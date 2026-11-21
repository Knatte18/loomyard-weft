---
verdict: APPROVED
findings:
  - id: F4
    severity: MEDIUM
    class: design
    carries: F4
    origin: lens:code-correctness
    location: internal/gateslot/tiers.go:473
    summary: MeasureCall includes a target when the only target-taking tier is covered and dropped, so planTierRuns refuses the rendered command.
    fix: In Config.MeasureCall in internal/gateslot/tiers.go, derive the target placeholder from the effective tiers after the same coverage elimination used by planTierRuns, sharing that selection logic with execution. Extend TestConfig_TierUse's MeasureCall table with a key listing target-taking tier0 and target-free tier1 whose Covers contains 0; assert a measuring call without the target placeholder.
  - id: F7
    severity: LOW
    class: design
    carries: F7
    origin: lens:code-tests
    location: internal/gatecli/tierrun_spawn_test.go:264
    summary: The result-argument assertion uses the production helper for its expected words, so removing the TRX logger or blame option leaves the test green.
    fix: In TestRunTierReadsTheVSTestResultsFile, replace vstestResultsArgs(resultsDir) in wantArgs with the literal words --results-directory, resultsDir, --logger, trx;LogFilePrefix=gate, and --blame, then append the user arguments. Keep the existing fixture and rows.
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
    answer: "Settled by the added decision titled \"Added after Discussion (parent, 2026-11-21): Drop the MeasureCall tier fallback\"; MeasureCall returns Gate's refusal and renders no fallback."
  - id: F3
    status: resolved
  - id: F4
    status: open
    answer: The missing-args, missing-tier0 and card-scope cases are fixed, but coverage elimination still makes a rendered target invalid.
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: open
    answer: The new rows cover invocation baselines, outcome mapping, fallbacks and passing messages; the logger and blame assertion still computes its expectation through the production helper.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
---

### [MEDIUM] Covered tiers can make the measuring command invalid

**Location:** internal/gateslot/tiers.go:473

**Issue:** A darn-write key lists tier0 with `dotnet test {targets} {args}` and tier1 with `dotnet test All.sln {args}`.
Tier1 declares `covers: [0]`.
`MeasureCall` sees tier0 and includes a target placeholder.
`planTierRuns` drops tier0 at `internal/gatecli/tiers.go:109`, then refuses explicit targets at line 147 because the remaining tier takes none.
This leaves prior F4's command-validity defect open.

### [LOW] Expected result arguments repeat the implementation

**Location:** internal/gatecli/tierrun_spawn_test.go:264

**Issue:** Both the production command and the expected argument list use `vstestResultsArgs`.
The fixture copies its TRX file using only the results-directory option.
Removing `--logger`, its value, or `--blame` changes both argument lists and leaves fixture production intact.
The test therefore does not pin the argument portion of prior F7.
The rubric's independent-expectation rule governs over the prior finding's suggested helper-based assertion.

## Verification

Reviewed the supplied committed diff, the board entry, the decision record, the focus cases, and the execution paths cited above.
The report-only write restriction prevented running a test gate that creates records and temporary fixtures outside that surface.
The supplied gate evidence was considered as advisory evidence; no test execution is claimed here.
