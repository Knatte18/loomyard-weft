---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/wait.go:408
    summary: The EndedEarly causes of the zero-attempts halt fix have no test beyond the no-input-box one
    fix: In internal/shuttleengine/gate_test.go add rows that finalize a gated run through the entry-wait-expiry path, the re-prompt-send-failure path and the plain session-end path, and assert GateOutcome.EndedEarly names each cause. Add rows asserting EndedEarly is empty on a pass, on a terminal failure and on a failure whose entry spent its attempts.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/darnwiring.go:452
    summary: Nothing pins that only the darn-write key gets --stop-on-fail on both its told and its tree-scoped command
    fix: In internal/loomcli/wiring_darn_test.go assert that VerifyCommand("darn-write") ends with the stop-on-fail flag, that VerifyCommand for another key such as the landing key does not carry it, and that VerifyTreeScope("darn-write") carries it while an empty scoped command stays empty.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomshed/verifygate.go:723
    summary: Nothing pins the budget clock wiring into NewVerifyGate or the verify_budget_min to Limit conversion
    fix: Add a loomshed gate-level test that passes NewVerifyGate a VerifyBudget with a short Limit, an injected clock and a recording Notify, runs a failing verify, and asserts Notify fires once with the attempt and latest failed run. Add a darnwiring assertion that cfg.VerifyBudgetMin of N gives Limit of N minutes in c.env.VerifyBudget.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/wait.go:2052
    summary: gateEndedEarly was inserted between the doc comment of finalizeAndRemoveDiedWindow and its func line, so that function has no doc comment and gateEndedEarly has two stacked ones
    fix: Move gateEndedEarly and its own doc comment above the finalizeAndRemoveDiedWindow comment, so the comment "finalizeAndRemoveDiedWindow finalizes the run with outcome..." sits directly above its func line again.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/verifygate.go:205
    summary: Failure branches of the failed-tests rerun, the stop-on-fail exclusions and two budget behaviours have no test row
    fix: Add rows to the existing tests for runFailedTestsAlone's dirty-tree result, a timed-out or crashed rerun keeping nothing, a rerun failure naming no test and the render-error fallback. Add rows to TestGateTest_Scenario for a build-blocked stop and a doubly crashed tier that does not stop the call. Add rows to TestVerifyBudget for the notice cut at verifyNoticeBytes on a rune boundary and the exit-code fallback of failed().
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: internal/loomshed/verifybudget.go:606
    summary: The budget notice names only the latest failed run, while the Goal asks for the gate's failures to date
    fix: In budgetClock keep one entry per failed verify run so far, attempt number with its failed runs and log, and render all of them in verifyBudgetNotice within the verifyNoticeBytes cap, or record in the decision record that the latest failure suffices.
---

### [MEDIUM] EndedEarly causes untested

**Location:** internal/shuttleengine/wait.go:408
**Issue:** Only the no-input-box cause is asserted, in `TestGate_RepromptOverPicker`. The wait-expiry, send-failure and session-end causes, and the empty result on a pass, a terminal failure and a spent entry, are unpinned. An unconditional `EndedEarly` would pass every current test, and this is the core of the issue #92 fix.

### [MEDIUM] Darn-Write-only stop-on-fail unpinned

**Location:** internal/loomcli/darnwiring.go:452
**Issue:** The Goal makes the verify gate default to stopping at its first failing tier. Dropping the wrapper, or applying it to the landing key, breaks no test.

### [MEDIUM] Budget wiring unpinned

**Location:** internal/loomshed/verifygate.go:723
**Issue:** `TestVerifyBudget` drives `budgetClock` directly, and every `NewVerifyGate` test passes `VerifyBudget{}`. Deleting `clock.arrive`, `clock.arm` or `clock.failed`, or the minutes-to-duration conversion in the wiring, fails nothing.

### [MEDIUM] Orphaned doc comment

**Location:** internal/shuttleengine/wait.go:2052
**Issue:** The diff leaves `finalizeAndRemoveDiedWindow` without its doc comment and gives `gateEndedEarly` two. Both the correctness and the affected-set lens found it.

### [LOW] Untested failure branches

**Location:** internal/loomshed/verifygate.go:205
**Issue:** The rerun's dirty-tree, timed-out, crashed, no-named-test and render-error branches, the build-blocked and doubly crashed stop-on-fail cases, and the notice cut and exit-code fallback have no row. They are filed once, as one coverage gap.

### [LOW] Notice names only the latest failure

**Location:** internal/loomshed/verifybudget.go:606
**Issue:** The Goal says to tell the orch "with the gate's failures to date". `budgetClock.latest` keeps one failure, so earlier attempts' failures are lost from the notice. The notice does name the log of the latest run, which reaches the rest, so the gap is small.

## Rejected

- Removal of the `-C names the run directory` line from the `gate test` help (lens:code-affected-set, LOW scope): the `-C/--dir` flag's own usage string still describes it, and commit bd60d136 names a help cap, so the removal is licensed and loses no information.
- Doubly crashed or build-blocked `--stop-on-fail` behaviour as a correctness defect: the correctness lens ran those cases and found them right; only the missing rows stand, in F5.
- A spent budget between `arrive` and `arm` delaying the notice to the next arrival, and `timer.Stop` letting a notice trail the gate's return: the correctness lens judged both minor and harmless, and the notice still goes out once.
- `verify_attempts: 0` keeping the old "after 0 attempts" wording: only a misconfiguration reaches it, and the decision record's case is the gate failing with attempts left.
- Test redundancy: the code-tests lens found no new test that duplicates another's coverage.
