---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_gate_test.go (TestFinalize_FlakyGateFailureLands)
    summary: "the test installs onFlakyPass itself, so the NewFinalize wiring that enables the whole feature has no test"
    fix: "In TestFinalize_FlakyGateFailureLands drop the line `fx.fz.gate.onFlakyPass = fx.fz.noteFlakyPass`, and have newFinalizeGateFixture build its Finalize through the wiring NewFinalize performs (or call NewFinalize), so deleting `fz.gate.onFlakyPass = fz.noteFlakyPass` in finalize.go turns the test red."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/verifygate_test.go (TestVerifyGate_RerunsATestFailureOnce)
    summary: "the rerun guards for a runner crash, a shell start failure and an empty log have no row, and a rerun that fails differently or is skipped is unpinned"
    fix: "Add rows to TestVerifyGate_RerunsATestFailureOnce. A first result with RunnerCrash true, one with ExitCode -1, and one with an empty Log each expect a single verify command and a failing reason. A second result with TimedOut true expects a timeout reason naming the second log. A second result with StatusSkipped expects a flaky pass. Removing the RunnerCrash guard in rerunTestFailure must turn a row red."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gateslot/runlog.go:358 (TierRunFailures), internal/landingshed/verifygate.go:300 (rerunTestFailure)
    summary: "a package that failed outside any test, such as a test-package compile error, counts as a test run and reruns the whole call, though it is deterministic"
    fix: "In TierRunFailures return ok false when a failure line ends in `, failed outside any test`, and add a TestTierRunFailures row for it (its current first row then needs that line removed). Alternatively state in landingshed/doc.go and the rerunTestFailure comment that such a package failure is rerun with the whole call."
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/gatecli/failurereport.go:265 and :294
    summary: "the failure report's producer writes `its output:` and `... N more lines cut` as literals while gateslot now declares them as reportOutputLine and reportCutPrefix for its parser"
    fix: "Export the two constants from gateslot (for example ReportOutputLine and ReportCutPrefix), and use them in internal/gatecli/failurereport.go in place of the literals `  its output:` and `... %d more lines cut`, so the producer and the parser of the format share one definition."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_gate_test.go, internal/gateslot/failedtests_test.go, internal/gatecli/counters_test.go
    summary: "remaining coverage gaps: the flaky section in the GitHub merge body and its dedup, TierRunFailures on an empty report or a colon-ended cause line, the VSTest line cap and result-line block close"
    fix: "In TestFinalize_FlakyGateFailureLands assert the body given to the GitHub merge contains flakyHeading, and add a flakySection case with two passes sharing a failure expecting one bullet. In TestTierRunFailures add a header-with-no-runs row (ok false) and a row with a cause line ending in a colon under a run. In the TestCounters VSTest rows add a block of more than failureLines lines and a Failed block directly followed by a Passed line with no blank between."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-gate-slot.md:105 and :108
    summary: "the gate-slot background describes Finalize's landing gate and the FailedTestsCall sites without the one rerun on the same head"
    fix: "In pattern/PATTERN-gate-slot.md add one sentence to the Finalize bullet saying a Finalize gate failure whose report lists tier and guard runs alone reruns once on the same head before any fixer, and that a passing rerun is recorded as flaky; and name that rerun as a second site of FailedTestsCall in the bullet at :105."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/finalize.go (Call doc, line about the flaky listing)
    summary: "the Call doc says only the landing commit lists the flaky failure, while the spec and package doc also name the GitHub merge"
    fix: "In the Call doc line `After a gate failure that passed when run once more on the same head, it then lists that failure as flaky.` name both the landing commit and the GitHub merge's body."
---

### [MEDIUM] NewFinalize wiring of the flaky sink is untested

**Location:** internal/landingshed/finalize_gate_test.go (TestFinalize_FlakyGateFailureLands)
**Issue:** The test assigns `onFlakyPass` by hand, so removing the assignment in `NewFinalize` disables the feature with the suite green.

### [MEDIUM] Rerun guards and rerun outcomes lack rows

**Location:** internal/landingshed/verifygate_test.go (TestVerifyGate_RerunsATestFailureOnce)
**Issue:** The runner-crash, `ExitCode < 0` and empty-log exits of `rerunTestFailure` have no row; the commit message states the runner-crash rule. A rerun that is skipped or fails differently is unpinned.

### [LOW] A package failure outside any test reruns the whole call

**Location:** internal/gateslot/runlog.go:358, internal/landingshed/verifygate.go:300
**Issue:** `TierRunFailures` accepts a `failed outside any test` line, so a deterministic compile or `TestMain` failure costs a second full gate run before the fixer, against the doc's "build failure is not rerun".

### [LOW] Producer of the report format keeps literals its parser now names

**Location:** internal/gatecli/failurereport.go:265 and :294
**Issue:** `gateslot` declares `reportOutputLine` and `reportCutPrefix` for parsing, while the producer repeats the strings, so a change of one silently breaks the other (`PATTERN-code-conventions`: one named constant).

### [LOW] Remaining coverage gaps

**Location:** internal/landingshed/finalize_gate_test.go, internal/gateslot/failedtests_test.go, internal/gatecli/counters_test.go
**Issue:** The merge body's flaky section, `flakySection` dedup, two `TierRunFailures` edge inputs and two VSTest block rules have no row.

### [LOW] PATTERN-gate-slot background is silent on the rerun

**Location:** pattern/PATTERN-gate-slot.md:105 and :108
**Issue:** The background states Finalize's gate and the `FailedTestsCall` sites without the new rerun; no PATTERN entry is declared changed, so this is a gap in the background only.

### [NIT] Call doc omits the GitHub merge

**Location:** internal/landingshed/finalize.go (Call doc)
**Issue:** The doc names the landing commit alone, while the spec and package doc also list the section in the merge.

## Rejected

- Rerun of the failed tests alone passes a test that fails only in combination (the parallel race): the board entry's expected behaviour licenses rerunning "the failed tests (or the failed tier)", and the record keeps the first failure's names in the flaky section.
- A VSTest error-message line starting with `Failed` and ending in `[...]` opens a bogus failed test: rare, and the cost is an extra named test in the report.
- Duplicated test coverage between `TestFinalize_FlakyGateFailureLands` and `TestVerifyGate_RerunsATestFailureOnce`: the first pins the landing text and no-fixer outcome, the second the gate's rerun rules; no overlap.
