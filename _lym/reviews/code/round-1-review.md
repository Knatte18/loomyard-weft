---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/webstercli/ownership.go:291
    summary: A transient lock-probe error refuses with run_ended true, so the Master stencil tells a live run's Master to end its turn although the refusal's own way forward is to re-run the verb
    fix: In ownership.go make orphanedMasterRefusal return the message and a runEnded bool that is true only for the free-lock case, and in refuseOrphanedMaster put run_ended true into the ErrFields map only when runEnded is true. Add a probe-error row to TestRefuseOrphanedMaster_RefusesAMasterWhoseRunEnded (for example run.lock is a directory) asserting the refusal carries the re-run way forward and no run_ended key.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/webstercli/cli.go:326
    summary: No test shows that addVerbs wraps each of the six bracket verbs and no other verb, so dropping a wrapper leaves every test green
    fix: Add a case to internal/webstercli/ownership_test.go that builds the verb tree through addVerbs with a free run.lock and a Master strand name, runs each bracket verb (begin-batch, await-batch, record-batch, recover-batch, fix-verify, fix-plan) and asserts a run_ended refusal, then runs a non-bracket verb such as status and asserts it does not refuse for that reason. Fold it into the existing test as rows, not a new test.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/websterengine/verifygatereport.go:261
    summary: A failed verify that names no test now has no Failures, so renderVerifyGateFindings writes a bare "Failing identities:" header with no entries before the log tail and the card hint
    fix: In renderVerifyGateFindings, before the final writeFailingIdentities call, add a branch for len(r.Failures) == 0 that writes one line saying the verify command exited with code r.ExitCode and named no failing test, then calls writeLogLocation and returns without the empty list and the card hint. Add a tail-only row to the findings-render test in verifygatereport_test.go.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: docs/overview.md:1164
    summary: The decision-add description and the verb's Long help list only notified and notice_errors and omit the new notice_held_back key
    fix: In docs/overview.md line 1164 and in the Long help text of the decision verb in internal/loomcli/decision.go (around line 1350), add one sentence saying a Webster Master session of a run that is not running gets no notice and is named under notice_held_back.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: design
    location: internal/websterengine/verifygate.go:448
    summary: Uncovered branches of the new stuck-reason code - the failedEntryReprompts fallback to gate.Attempts, cutHead and the rune-boundary loops of cutHead and cutTail, the per-arrival command reset, and a failure with named tests keeping Failures with no LogTail
    fix: Add rows to the existing tests in verifygate_test.go. In TestVerifyGateStuckReason_CountsRepromptsAgainstTheLimit add a row whose GateOutcome has no failed entry. In TestVerifyGateStuckReason_NamesTheFailureAndAWayForward add a run-level Findings row longer than verifyGateStuckTextBytes and a LogTail row with a multibyte rune straddling the cut. Add a gate-level row that fails on a dirty tree after an earlier arrival rendered a command and asserts Command is empty, and one with named failing tests asserting Failures is kept and LogTail is empty.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: design
    location: internal/websterengine/verifygate.go:491
    summary: A failure whose output was empty renders an output tail clause with nothing after it
    fix: In the default branch of verifyGateStuckFailure, when report.LogTail is empty, render "its output was empty" in place of the "output tail:" clause.
    origin: lens:code-correctness
---

### [MEDIUM] Transient probe error tells a live Master to quit

**Location:** internal/webstercli/ownership.go:291
**Issue:** `orphanedMasterRefusal` refuses a Master strand when `RunActive` fails, with the way forward "re-run the verb". `refuseOrphanedMaster` puts `run_ended: true` on every refusal, and the Master stencil says to end the turn with no tool call on that key. A filesystem hiccup therefore ends a live run's Master session, against the refusal's own text.

### [MEDIUM] Bracket-verb wiring is untested

**Location:** internal/webstercli/cli.go:326
**Issue:** The only test calls `refuseOrphanedMaster` on a synthetic command. The decision record's expected outcome, that `record-batch` without the lock refuses, depends on `addVerbs` wrapping the real verbs, and nothing fails if a wrapper is dropped.

### [LOW] Empty "Failing identities" list for a no-test failure

**Location:** internal/websterengine/verifygatereport.go:261
**Issue:** The gate sets `Failures = nil` and `LogTail` for a failure that named no test. The renderer still writes the `Failing identities:` header, then the log tail and the "No recorded card touched a failing package" hint. The fixer reads an empty list.

### [LOW] notice_held_back is missing from the docs and the verb's help

**Location:** docs/overview.md:1164
**Issue:** The envelope gained `notice_held_back`. `internal/loomcli/doc.go` and the websterengine `doc.go` carry it. `docs/overview.md` and the verb's `Long` help still list only `notified` and `notice_errors`.

### [LOW] New stuck-reason branches with no row

**Location:** internal/websterengine/verifygate.go:448
**Issue:** `failedEntryReprompts` falling back to `gate.Attempts`, `cutHead`, the rune-boundary loops, the per-arrival `command` reset and the named-failures case of the gate each have no test row that would fail if the behavior broke.

### [NIT] Empty output tail renders as a dangling clause

**Location:** internal/websterengine/verifygate.go:491
**Issue:** With empty verify output the reason reads "output tail: ; see <log>".

## Rejected

- Issue 97 stencil change: step 6 makes a fix after a card commit land as a new commit and forbids amend, squash, rebase and reset. The Master stencil keeps every existing rule. The decision record offers batten's leased force-push only as a fallback, and the diff adds none. No defect.
- `record-batch` without the lock refuses only for a Master strand and warns for other callers. The decision record's expected outcome is "refuses rather than warns", but the file header gives the sandbox suite's manual driving as the reason. Master is the caller that caused the fault, so I take the split as in scope.
- The `(fix)` subject suffix against code that reads commit subjects: gate-fixer commits are matched by the `fix:` prefix, which a card subject ending in ` (fix)` does not match. No defect.
- The code-correctness fork's open question, whether a Master in the `waiting` rung sits in a non-running shed state: the stencil has Master wait while `lym webster run` is live, so the state is running. Not a defect.
- Attempt count against the limit in the halt reason: `verifyGateStuckLead` counts the failing entry's re-prompts against `report.Cap`, and the zero-failure and exact-limit rows agree. The halt reason names the command, the failure or the tail, and ends in a way forward, as 1b6e3fca does for darn. No defect.
- Test economy: the three new tests each pin a distinct behavior, and the redundancy report judged none of them. No finding.
