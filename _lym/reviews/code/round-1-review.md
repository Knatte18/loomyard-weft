---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/sheddriver/agentshuttle.go:21
    summary: "Goal asks for the StopFailure re-send for any role run under a deadline, but only the arbiter and tackler specs set APIErrorRetries, so every other deadline-bound role still idles to its deadline on a StopFailure."
    fix: "Decide per role under a deadline, and set Spec.APIErrorRetries where the role has an owner that can act on api-error. At least the shuttle.Spec constructions that carry a Timeout and an autonomous agent need it, such as the gate fixer in internal/gatefixer/spawn.go, the conflict resolver in internal/mergeresolve/spec.go, the seat reviewers in internal/seatengine/prompt.go and the burler, treadle, landing and webster specs. Each must map OutcomeAPIError in its own outcome switch, since a caller that only tests for OutcomeDone reads it as one more failure. Where a role is left out on purpose, state the reason once beside the retry constant, and ask the run's parent to record the narrowing with lym loom decision add, because the Goal names any other role run under a deadline."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/carryover.go:118
    summary: unfinished does not treat a recorded api-error outcome as an unfinished run, so the next settler attempt of the role drops the previous attempt's carried and operator instructions.
    fix: "In internal/shuttleengine/carryover.go add string(OutcomeAPIError) to the case list in unfinished, beside runOutcomeStopped, OutcomeDied and OutcomeTimeout, and update the doc comment above it to name an api-error end. Add an api-error row to the existing table-driven test that covers unfinished."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/orch-role.md:135
    summary: The orch role contract lists the hand-on causes a review escalation can carry and omits the new api-error cause, so the orch can read the new hand-on as outside the listed fault cases.
    fix: "In contracts/orchrole/orch-role.md line 135 add `api-error` to the cause list in parentheses, after `precondition`. Say nothing else there."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/sheddriver/tackler_test.go:283
    summary: The tackler api-error row asserts cause, verdict and stop count only, so reverting HandBackQuestionCause at tackler.go:440 leaves every test green although the parent's notice naming the error is the Goal's second Expected point.
    fix: "In TestTacklerCall_SpawnPath add a wantQuestion field to the case struct, assert strings.Contains on the envelope's Question when it is non-empty, and set it to server_error on the new api-error row. Add no new test function."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/apierrorretry_test.go:419
    summary: No row of TestWait_APIErrorRetry reaches the clear of pending by a later turn start, turn end or session start, so deleting that case in apierrorretry.go:333 leaves every row green while a recovered session would be re-sent a line.
    fix: "Add a row to the TestWait_APIErrorRetry table where the events file gains START and STOP lines by itself after the API error with the output file still missing, then the run is ended by a short timeout, and the row asserts no re-send was typed. Reuse the table, add no new test function."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/apierrorretry.go:312
    summary: A usage-limit StopFailure is re-sent three times and then handed on as api-error, although the held-turn path sends the parent no notice while the account is limited because the parent cannot act before the reset.
    fix: "In retryAPIError consult the limit reading the held-turn path uses before re-sending, and for a limited account end the run at once with the limit text in Result.APIError, or document the bound in the retryAPIError doc comment if re-sending a limited account is accepted."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedarbiter/call.go:269
    summary: When a StopFailure payload carries no error text, result.APIError is empty and the reason and question end in a dangling colon, so the parent is told no error.
    fix: "Give an empty Result.APIError a fixed fallback text such as unknown error in endOnAPIError in internal/shuttleengine/apierrorretry.go, so both settlers and HandBackQuestionCause always carry text. Add one row to TestWait_APIErrorRetry with an APIERR line with no text."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/apierrorretry.go:347
    summary: A re-send that fails logs at Info on every poll tick until the deadline, where the loop's other failing sends log once per episode or drop to Debug on repeats.
    fix: "In retryAPIError log the first failed send at Info through run.failures.Fail with a key of its own, call run.failures.Recover on a delivered send, and log repeats at Debug, as handleGatedBoundary does for its pending send."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-shuttle-provider-seam.md:54
    summary: The Completion Signal entry lists the negative answers and the helper sites without OutcomeAPIError and retryAPIError, which the tripwire test now audits, so the text and the test disagree.
    fix: "In pattern/PATTERN-shuttle-provider-seam.md add OutcomeAPIError to the list of negative answers at line 54, and name retryAPIError among the helpers through which allOutputFilesExist is reached at line 56."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md
    summary: The shuttle bullet states that completion is classified into three outcomes done, died and timeout and that a held turn end always waits to the deadline, which the new api-error outcome and the APIErrorRetries opt-in contradict.
    fix: "In the shuttle bullet of docs/overview.md name api-error as a fourth outcome, and add one sentence that a spec with APIErrorRetries re-sends a turn the provider's API ended and then ends the run api-error."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/apierrorretry.go:338
    summary: Three branches of retryAPIError have no row, the pause that covers the run, a failed send retried on a later tick, and a run attached over an events file that already holds earlier API errors, so none fails if broken.
    fix: "Add rows to the TestWait_APIErrorRetry table, one with a covering pause record that re-sends and ends nothing, one whose first SendText fails and whose second succeeds with one delivered re-send, and one that seeds several earlier APIERR lines and ends api-error with no re-send. An engine without SessionSignalParser needs no row."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/attach.go:615
    summary: The recorded api-error outcome is pinned by no attach row and no session-state shadow row, so dropping it from isTerminalOutcome or loopAgreesWithState leaves the tests green.
    fix: "Add a string(OutcomeAPIError) row to the existing table-driven test of isTerminalOutcome or the attach disposition, and an api-error loop-classification row to the existing waitshadow table that expects no disagreement."
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: internal/shuttleengine/apierrorretry.go:341
    summary: retryAPIError calls classifyDeadlineExpiry twice in a row for one decision.
    fix: "In retryAPIError compute outcome := run.classifyDeadlineExpiry(OutcomeAPIError) once before the condition and pass it to endOnAPIError, so the condition reads watch.errors > run.spec.APIErrorRetries || outcome == OutcomeDone."
---

### [MEDIUM] Roles under a deadline other than arbiter and tackler are left silent

**Location:** internal/sheddriver/agentshuttle.go:21, with the spec constructors named in the fix.
**Issue:** The Goal's Expected section asks that a StopFailure of a settler's session, "arbiter, tackler, and any other role run under a deadline", be read as the end of the turn and re-sent a bounded number of times. The diff opts in the arbiter and tackler specs only, and the change description says only the two settlers opt in. Every other autonomous role with a Timeout, such as the gate fixer, the conflict resolver, the seat reviewers and the burler and landing agents, still sits idle to its deadline on a StopFailure. No decision record entry narrows the Goal to the two settlers, so the narrowing is the writer's own.

### [MEDIUM] carryover treats an api-error end as finished

**Location:** internal/shuttleengine/carryover.go:118
**Issue:** `unfinished` names stopped, died and timeout as the unfinished outcomes and lets api-error fall through to false. `attach.go` records api-error as a terminal outcome, and each settler attempt after it starts under the same role key. The new attempt therefore drops the carried and operator instructions that a timed-out or dead predecessor would have handed on.

### [MEDIUM] The orch role contract omits the new hand-on cause

**Location:** contracts/orchrole/orch-role.md:135
**Issue:** The contract lists the causes a hand-on can carry as timeout, died, missing, unparseable and precondition, and its sentence defines which hand-ons the orch handles as faults. The diff adds a hand-on with cause api-error that reaches the orch, and the list does not name it.

### [MEDIUM] The tackler's question naming the error is not asserted

**Location:** internal/sheddriver/tackler_test.go:283
**Issue:** The tackler's `handBack` builds its question through `HandBackQuestionCause`, which is how the parent's notice names the error and the failed settler. The tackler's new row asserts cause, verdict, starts and stops only. The arbiter's row asserts the question, and the tackler's has no counterpart, so reverting the one changed line at tackler.go:440 passes every test.

### [MEDIUM] The clear of a pending API error is not pinned

**Location:** internal/shuttleengine/apierrorretry_test.go:419
**Issue:** `pending` is cleared by a turn start, turn end or session start read after the API error. Row 2 ends with an APIERR line that sets `pending` again, and rows 1 and 4 never read a later turn signal. Removing that case in apierrorretry.go leaves every row passing, and a session that recovered by itself would then be sent a line it does not need.

### [LOW] A usage-limit StopFailure is retried like a server error

**Location:** internal/shuttleengine/apierrorretry.go:312
**Issue:** The held-turn path sends the parent no notice while the account is limited, because the parent cannot act before the reset. `retryAPIError` does not read that state, so a limited account is re-sent three times over about 210 seconds and then handed on as api-error, a stop the parent cannot clear.

### [LOW] An API error without text leaves a dangling colon in the notice

**Location:** internal/shedarbiter/call.go:269
**Issue:** `apiErrorText` returns the empty string when the payload carries none of its three keys. The reason then ends in a colon and the question reads `(api-error: ...: )`, so the parent is told no error although the Goal asks for it named.

### [LOW] A failed re-send logs at Info on every tick

**Location:** internal/shuttleengine/apierrorretry.go:347
**Issue:** `pending` stays true after a failed send, so each poll tick logs the same Info line until the deadline. The gate send in `handleGatedBoundary` logs once per episode and drops repeats to Debug, which is the level policy a loop log follows.

### [LOW] The Completion Signal entry does not name the new negative answer

**Location:** pattern/PATTERN-shuttle-provider-seam.md:54
**Issue:** The entry enumerates the negative answers and the helper sites that reach `allOutputFilesExist`. The tripwire test now audits `OutcomeAPIError` and `retryAPIError`, and the entry's text names neither.

### [LOW] docs/overview.md still states three outcomes

**Location:** docs/overview.md, the shuttle bullet.
**Issue:** The bullet says completion is classified into done, died and timeout and that a held turn end waits to the deadline. The diff adds a fourth outcome and a spec opt-in, and touches no file under docs.

### [LOW] Three branches of retryAPIError have no row

**Location:** internal/shuttleengine/apierrorretry.go:338
**Issue:** No row sets a pause that covers the run, fails a send, or attaches over an events file that already holds earlier API errors. The change description claims a re-attach resumes the count from the events file, and nothing pins it.

### [LOW] The recorded api-error outcome is pinned by no attach or shadow row

**Location:** internal/shuttleengine/attach.go:615
**Issue:** `isTerminalOutcome` and `loopAgreesWithState` each gained an api-error case, and neither `attach_test.go` nor `waitshadow_test.go` is in the diff. Dropping either case leaves the tests green; the shadow is display only.

### [NIT] Duplicate classifyDeadlineExpiry call

**Location:** internal/shuttleengine/apierrorretry.go:341
**Issue:** The condition and the call it guards each run `classifyDeadlineExpiry(OutcomeAPIError)`, so one decision reads the output files twice.

## Rejected

- Retry budget counts every API-error turn end since the prompt offset rather than consecutive ones: the change description states it as a design decision, and the Goal asks only for a bounded number of re-sends.
- A re-sent turn that produces no hook signal leaves the run waiting to its deadline: the send is verified, and the deadline still hands the stop on, so the run is not silent past it.
- No row for an engine without SessionSignalParser or an unreadable events file: both are silent no-ops of an opt-in capability, and `pollEventsTick` owns the read-failure count.
