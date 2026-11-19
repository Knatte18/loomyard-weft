---
verdict: APPROVED
findings:
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/mutation.go#mutationStep.recordFindings
    summary: Failed mutation calls still discard completed packages' findings before the report receives them.
    fix: >-
      In internal/gatecli/mutatecheck.go, return the accumulated outcome from diffMutation.run on failure,
      including completed results and the current run count.
      In internal/gatecli/mutate.go, assign the returned outcome to found.outcome before checking the error.
      Keep recordFindings from recording Functions or FileHashes on failure.
      Extend the mutation execution coverage with a failure after an earlier package produced findings,
      asserting that those findings reach the report alongside the error.
    origin: lens:code-correctness
    carries: F2
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/gatecli/mutation.go#mutationStep.recordFindings
    summary: No report-step test asserts persisted unjudged mutants or packages not judged.
    fix: >-
      Extend TestGateMutate's report-step coverage in internal/gatecli/mutate_integration_test.go
      with nonempty unjudged-mutant and not-judged-package results.
      Read the written report with ReadMutationReport and assert both lists and their reasons.
      Use the injectable budget units to exercise the unjudged case without waiting out a production budget.
      Keep the existing survivor and uncovered-block assertions.
    origin: lens:code-tests
    carries: F5
prior:
  - id: F1
    status: resolved
    answer: mutationStep.run asks Config.HasGoCoverageMaps, which reads the scoper-keyed capability table.
  - id: F2
    status: open
    answer: >-
      Failed calls now omit Functions and FileHashes, addressing the stale-match case with changed functions.
      The recorder's synthetic partial-count test bypasses the production paths that still discard partial results.
  - id: F3
    status: resolved
    answer: gateslot/doc.go names the mutation report step's slot ownership, shared bounds, and even cap spread.
  - id: F4
    status: resolved
    answer: >-
      TestGateMutate covers red gate, no hub, unsupported scoper, and no parent reports and their summary lines.
      Its red-tier call also checks that exit code 1 remains unchanged.
  - id: F5
    status: open
    answer: >-
      The new report row asserts survivors and uncovered blocks.
      Unjudged mutants and packages not judged remain asserted only through the hand verb, not mutation.json.
  - id: F6
    status: resolved
    answer: >-
      The integration row checks rejection after a changed function.
      The tier0 table checks note, red-gate, and prime refusals.
  - id: F7
    status: resolved
    answer: >-
      logsOnly rejects conditions containing calls.
      The added PlanLineMutants row asserts that such a condition retains its mutant.
  - id: F8
    status: resolved
    answer: PlanLineMutants rows cover else, else-if, initializer, reversed operands, and shadowed runtime.
  - id: F9
    status: resolved
    answer: The review-note table now covers a non-red judge-nothing note and a HEAD-match error.
  - id: F10
    status: resolved
    answer: The description and selection comment describe sampling functions rather than promising every package a share.
  - id: F11
    status: resolved
    answer: The report type, matching function, and package documentation state the non-Go-file freshness bound.
---

### [MEDIUM] Completed mutation findings disappear on a later failure

**Location:** `internal/gatecli/mutation.go#mutationStep.recordFindings`

**Issue:** Suppose package A produces a survivor and package B's required binary build or test start fails.
`diffMutation.run` returns an empty outcome on the error at `internal/gatecli/mutatecheck.go:200`.
`mutationCall.run` also returns before assigning the outcome at `internal/gatecli/mutate.go:299`.
The recorder therefore receives none of A's results.
Its test supplies partial counts directly and cannot detect either loss.
This carries the unresolved partial-findings portion of F2; the error-path identity omission is fixed.
The correctness and affected-set lenses independently identified this same defect.

### [MEDIUM] Written unjudged lists lack assertions

**Location:** `internal/gatecli/mutation.go#mutationStep.recordFindings`

**Issue:** The report row at `internal/gatecli/mutate_integration_test.go:547` checks survivors and uncovered blocks.
The unjudged and not-judged assertions elsewhere inspect the hand verb's answer.
`TestMutationStep_RecordFindings` supplies neither list.
Deleting the corresponding report assignments would therefore leave these tests passing.
This carries the remaining coverage gap in F5.

The review covered the committed range from `5823c67e3bb8bc2e9aab7aed301ffcb08a3f424c` to HEAD.
The board entry and Sketch agree on the requested scope.
Every lens returned; their reports are saved beside this review.
No new BLOCKING finding was identified.
The APPROVED verdict follows the format's severity rule and does not close the carried MEDIUM findings.

Package-scoped `go build` passed for gatecli, gateslot, testcover, loomcli, shedadapters, and shedrecipe.
Tests were inspected but not rerun in this review.
