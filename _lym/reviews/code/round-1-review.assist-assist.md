---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: pattern/PATTERN-test-speed.md:225
    summary: The scheduling change leaves the governing background and slot documentation describing sequential reports.
    fix: "Update pattern/PATTERN-test-speed.md's report scheduling descriptions at lines 225, 329 and 344 and internal/gateslot/doc.go at lines 45 and 79 to distinguish judgment beside the highest tier from publication after lints. Preserve the slot requirements and reach/redundancy/mutation ordering."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/gatecli/test_integration_test.go:1537
    summary: No regression assertion distinguishes concurrent report scheduling from the former sequential tail.
    fix: "Extend the gate scenario with controlled synchronization that holds the highest tier open and observes report work before releasing it. Assert no report work during an earlier tier or after that tier fails, and cover a single-tier plan. Use signals or controlled fixtures rather than elapsed-time comparisons. Exercise runTest's scheduling through gate behavior, not separate tests of lastTierStart or reportStepsOf."
    origin: lens:code-tests
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/gatecli/reports_test.go:36
    summary: The stopped-chain test waits for completion first and never exercises cancellation or joining active work.
    fix: "Extend the existing chain and gate lifecycle coverage with report work held until cancellation. Stop while that work is active and prove shutdown waits for its completion. Exercise the owner's tier-failure and halt paths, checking red-gate replacement after failure and no report publication after a tier halt. Do not call chain.wait before the stopped-chain action."
    origin: lens:code-tests
  - id: F4
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun_test.go:267
    summary: The report guard's production command registration and removal have no behavioral assertion.
    fix: "Extend the existing tier memory-guard coverage with a controlled live tier command and the shared report guard registry. Assert that its test descendants are excluded while report-owned runaways remain protected, then that command completion removes the exclusion. Exercise production registration, removal and guard wiring rather than manually populating the registry or testing its map helpers alone."
    origin: lens:code-tests
---

### [BLOCKING] Scheduling rules were left stale

**Location:** pattern/PATTERN-test-speed.md:225

**Issue:** The diff starts report judgment at `internal/gatecli/run.go:196`, before the highest tier runs.
The PATTERN background still places redundancy after slot release and reach and redundancy after lints.
`internal/gateslot/doc.go:45` and `:79` repeat those ordering assumptions.
The changed template and field documentation now disagree with these untouched descriptions.
The review instructions require BLOCKING severity for a listed constraint violation.
The amended Goal authorizes concurrency; the missing change is to its affected rule descriptions.

### [BLOCKING] Concurrent scheduling is not pinned

**Location:** internal/gatecli/test_integration_test.go:1537

**Issue:** The renamed scenario waits for completion and checks exit status, report fields and a summary line.
Those assertions also pass with the former sequential implementation.
`TestReportChain` calls `startReports` directly, bypassing the highest-tier and earlier-success conditions.
The reach report's unmapped helpers alone are not evidence of a defect;
the absence of any ordering assertion establishes the uncovered behavior.
This falls under the rubric's test-economy criterion and `PATTERN-test-speed`.

### [BLOCKING] Active shutdown is not pinned

**Location:** internal/gatecli/reports_test.go:36

**Issue:** Both test rows call `chain.wait()` before the stopped row calls `chain.stop()`.
The outside-hub fixture performs no report work, so that row proves only stopping an already completed chain.
It cannot detect loss of active cancellation, joining or the owner's cleanup on a tier halt.
The standalone redundancy cancellation case explicitly writes its result and does not cover the new owner lifecycle.
This is a separate uncovered behavior under the rubric's test-economy criterion and `PATTERN-test-speed`.

### [BLOCKING] Memory-guard ownership wiring is not pinned

**Location:** internal/gatecli/tierrun_test.go:267

**Issue:** The new `TestMemoryGuard` row populates its registry manually at `memoryguard_test.go:185–189`.
`TestRunTierMemoryGuard` constructs a request without that registry.
Neither assertion detects losing registration at `tierrun.go:534`, removal at `:558`, or wiring at `reports.go:92`.
The filter algorithm is covered, but its production connection to concurrently running tier commands is not.
This is uncovered behavior under the rubric's test-economy criterion and `PATTERN-test-speed`.

## Review coverage

Reviewed the supplied diff pieces and commits from `4ff6aae9c3128166e0d099a5b6943cbdaeb82129` through HEAD.
The board Goal and its added decision permit concurrent reports and remove the before-and-after timing requirement.
The highest-configured-tier implementation fits that Goal without hard-coding tier 2.
Red-first implementation and wiring are unchanged.
No unsupported new timing measurement or record-format change was identified.

The chain copies its execution inputs before starting, buffers step output and joins before consuming it.
Registry operations use a mutex, and the normal command-return path removes registered processes.
The guard's own output has a separate buffer and is joined before publication.
The ordering of guard lines is grouped rather than chronological; no machine consumer requiring chronology was found.
Map refresh retains its clean-tree checks, and mutation builds use overlays.
No new source mutation or report-file format conflict was established.
Lint failures retain the former pre-lint definition of a red gate.

No build or test was run: the review's write surface permits only the review and lens reports.
The supplied current-tree gate evidence was used without treating it as proof of missing behavioral assertions.

## Rejected

- Cancellation during the final report wait can still publish reports, but the base sequential tail does likewise.
  This is pre-existing behavior, not a finding against this diff.
- Missing before-and-after timing is authorized by the added decision and is not a scope finding.
- No duplicate or tautological new test was established; the reported gaps concern unasserted orchestration behavior.
