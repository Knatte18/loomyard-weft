---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/parentseat.go:199
    summary: The code-review ask omits the current round's reports and never covers the final round before Bouncer judgment.
    fix: Move the code-specific parent checkpoint after the current round's review and fixer report exist. Name both current artifacts in the request and consume the parent's satisfied answer or findings before that round's Bouncer judgment. Preserve the discussion checkpoint separately and update the code-seat tests and documentation.
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/driftengine/config.go:114
    summary: The requested drift_judge_ask_parent key is absent.
    fix: Expose drift_judge_ask_parent as the drift module's YAML setting and update its category entry, template, readers, config tests, registry expectations and documentation consistently. Apply F4's classification when exposing the renamed key.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/judgeparent.go:165
    summary: A restarted judge can store replacement rulings using approval of an earlier proposal.
    fix: Bind each ask to the proposal it presents and retain that association across judge restart. Require renewed agreement for replacement rulings that the old answer did not approve, and require the parent's requested changes to be applied before acceptance. Replace the modification-time-only check and add a restart regression to the existing producer test.
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/driftengine/config.go:164
    summary: Parent approval switches and bypassing timeouts are classified None despite the Guard rule.
    fix: Classify both new ask switches and both parent timeout keys as CategoryGuard in driftengine.ConfigSpec and loomengine.ConfigSpec. Update config registry expectations and documentation, and explain the task-local override departure with a Pattern-Departure trailer naming PATTERN-config-layers.
  - id: F5
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/driftengine/judgeparent_test.go:221
    summary: The retained contained tests lack the required testtiming keep annotations.
    fix: Add a testtiming keep directive immediately above TestJudgeAskLine and TestParentAskLine naming their independent exact-line assertions, and above TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments naming its seat-identity assertion. Retain the tests.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/judgeparent.go:138
    summary: Failed persistence of a no-parent transition is converted into a passing agreement gate.
    fix: Return the EndParentAsk persistence error from postJudgeAsk through sendJudgeAsk and agreementGate. Decide agreement from the persisted transition result instead of inventing a local no-parent state. Add a failed-transition row to the existing gate test.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentseatwiring.go:76
    summary: Wiring tests check seat presence but not the production behavior the new assignments supply.
    fix: Extend TestWire_ReviewKeysReachTheEnv to check the constructed seats' tags, subjects, per-round policies, discussion history lookup, selected setting keys and timeout selection using distinct configured values. Assert the parent fields before clearing them for the existing comparison.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftengine/verify.go:818
    summary: No producer-level assertion proves the parent agreement gate runs before rulings are stored.
    fix: Extend TestDriftVerify with a parent seat and its fake gate-loop runner. Assert that an open ask prevents storage and an answered ask permits it through the actual table source and gate execution. Cover Verify and Recheck and a held ask after the key is disabled, folding the private helper coverage into these rows.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftengine/judgeparent.go:162
    summary: The gate test omits pending-post recovery, boundary failures and a parent answer racing expiry.
    fix: Extend TestParentAgreementGate with pending-post replay, resolver failure, unreadable ask/report, failed initial record write and an answer arriving between the open-ask read and expiry. Use the injected clock callback to install the racing answer deterministically, and assert unchanged rulings remain blocked when that answer has findings. Assert the persisted state and post destination in recovery rows.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/parentreview.go:332
    summary: The generalized answer command still directs stale code and drift answers to review the decision record.
    fix: Make parentAnswerRefusalText direct the caller to the latest ask's named target instead of the decision record, and update the existing stale-answer assertion in parentreview_test.go.
---

### [BLOCKING] Ask about the current code-review round

**Location:** internal/shedadapters/parentseat.go:199
**Issue:** The board entry requires the round's review and fixer report before the Bouncer judges that round.
The ask runs before review starts, and round one names no reports.
Later asks name only the previous round's reports.
The final round therefore has no parent checkpoint covering its rulings and fixes.
The call order is visible in `burlerreview.go:375` and `burlerreview.go:555`.
The landing summary acknowledges the departure; no decision amendment authorizes it.
This consolidates the correctness and scope lenses' duplicate finding.

### [BLOCKING] Supply the requested drift key

**Location:** internal/driftengine/config.go:114
**Issue:** The goal names `drift_judge_ask_parent`.
The loader, template and category map instead expose `judge_ask_parent`.
The summary's naming rationale is a writer claim, and no amendment changes the requested name.

### [BLOCKING] Bind agreement to the rulings reviewed

**Location:** internal/driftengine/judgeparent.go:165
**Issue:** A judge can die after its parent approves proposal A but before storage.
The open session retains its spawn ID.
With no live chair, `MultiLLMProducer.Call` archives the output and starts a fresh judge.
That judge can produce B under the same spawn ID.
The agreement gate finds A's done ask and accepts any satisfied report without comparing its proposal.
A report with findings fares no better: a new output's later modification time satisfies the entire check.
The parent has neither approved B nor necessarily had its corrections applied.

### [BLOCKING] Respect the written config boundary

**Location:** internal/driftengine/config.go:164
**Issue:** `PATTERN-config-layers` classifies review/approval switches and pass-on-expiry timeouts as Guard.
The new drift and loom controls are classified None, which opens them to local overrides.
This concerns the meaning of their classifications, not the mechanical presence check.
The task asks for task-level settings, but the review directive explicitly gives the listed constraint precedence.
The existing discussion classification supplies no written exception.
The fix must record that conflict instead of silently choosing the task-local behavior.

### [BLOCKING] Mark distinct contained tests for retention

**Location:** internal/driftengine/judgeparent_test.go:221
**Issue:** Both request-line tests independently assert exact text.
Their covering producer tests build expected text with the formatter being tested.
The segment-placement test checks seat identity, while its covering wiring test checks only presence.
These are distinct assertions, so deletion is unwarranted.
The rubric and `PATTERN-test-speed` require a keep annotation for each retained contained test.

### [MEDIUM] Propagate failed ask-state persistence

**Location:** internal/driftengine/judgeparent.go:138
**Issue:** If posting fails and recording no-parent also fails, the code logs the second error.
It then returns a local no-parent ask and passes the gate.
The stored ask remains pending-post while the row may store rulings and close its session.
The answer and await commands consequently observe a different lifecycle from the producer.

### [MEDIUM] Assert production seat behavior

**Location:** internal/loomcli/parentseatwiring.go:76
**Issue:** The wiring test clears each parent seat after checking its presence.
The placement test constructs seats itself, and the adapter test sets the every-round policy itself.
None checks the production assignments that the mutation report deletes at lines 62–64 and 76–78.
The tests therefore miss changes to the actual tags, subjects and round policy.

### [MEDIUM] Exercise agreement through the drift producer

**Location:** internal/driftengine/verify.go:818
**Issue:** The new tests call the agreement helper or gate closure directly.
They do not establish that the producer appends and runs the gate before storage.
Removing the append or restoring the old table source would preserve those assertions.
Executing the real gate path through the fake runner also checks that the attempt budget enables the gate,
without requiring a test to pin an arbitrary retry count.

### [MEDIUM] Cover recovery and the answer-expiry race

**Location:** internal/driftengine/judgeparent.go:162
**Issue:** The gate test separately covers an already-done ask and an unanswered expired ask.
It never records findings between the open-ask read and the expiry transition.
The branch preserving such an answer can be inverted without an assertion failing.
Pending-post replay and the listed read, write and resolution failures also have no rows.
These are boundary behaviors of the new gate, and they fit its existing table.

### [LOW] Name the latest ask's target in recovery

**Location:** internal/loomcli/parentreview.go:332
**Issue:** The command now accepts code and drift asks.
Its stale-answer refusal at lines 165–168 still says the decision record may have changed.
For a latest drift ask, the caller must review proposed rulings instead.
The existing test retains the discussion-specific wording.

## Rejected

- Redundancy as grounds to delete the three contained tests: their independent assertions differ from their covers.
- Findings based only on the warning-condition mutants at judgeparent.go:138 and :144: negation changes logging only.
  F6 concerns failed persistence and a passing result, which is a separate observable failure.
- An additional scope finding for the orch-role, overview, CLI help or package-doc changes: they document the requested features.
- An additional finding merely because the drift deadline lacks the code-review cap: the task requires a timeout,
  and the existing restart path can carry the outstanding ask.
  F3 addresses the concrete approval defect in that restart path.

## Validation

`go build ./internal/driftengine ./internal/loomcli ./internal/loomengine ./internal/shedadapters` passed.
No test suite was run in this review.
The supplied reach, redundancy and mutation reports informed the coverage review.

