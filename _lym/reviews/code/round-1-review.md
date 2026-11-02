---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gitexec/gitexec.go:55
    summary: No test runs the production startStallTimer default, so a wrong bound, a check that never fires or a broken stop passes every row
    fix: In internal/gitexec/export_test.go add an exported wrapper that calls the package's default startStallTimer value (capture it in an unexported var at init so it stays reachable while SetStallCheckForTest has replaced the seam). Add one row to TestRun_RemoteBounds in internal/gitexec/gitexec_unix_test.go that starts it with a short bound and a check that signals a channel, waits on that channel with a 10 s guard timeout, and asserts the check ran once; then starts it with a one-hour bound, calls the returned stop, and asserts stop returned true. Use no fixed sleep.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/gitexec_unix_test.go:219
    summary: The silent row's readiness count >= attempt assumes one startStallTimer call per logical attempt, but runSpawning re-runs runAttempt on an EAGAIN start failure, so attempt then outruns the script's run count and the row waits out its 10 s deadline and fails
    fix: In internal/gitexec/gitexec_unix_test.go make the silent row's ready independent of the attempt number. For example keep a closure variable per armed attempt, the run count seen the first time ready is called for that attempt number, and report true once the count exceeds it; or change SetStallCheckForTest in export_test.go to hand ready a snapshot taken when the timer is armed. Keep the assertion that two checks were armed.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/gitexec_unix_test.go:283
    summary: The answering row never asserts that its check ran, so if git never wrote the trace the row still passes as a plain deadline kill and proves nothing about sparing an answering remote
    fix: In the answering row of internal/gitexec/gitexec_unix_test.go have the ready closure set a captured atomic.Bool when it returns true, and after remoteAttempts add an assertion that the flag is set, with a message saying the stall check never ran.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/gitexec/gitexec_unix_test.go:272
    summary: The armed() assertion in the unarmed row repeats what the row's Stalled == false, runs == 1 and one-kill assertions already catch
    fix: In the unarmed row of internal/gitexec/gitexec_unix_test.go delete the armed() assertion; keep SetStallCheckForTest with the always-true ready, which is what makes an armed check fail the existing assertions, and drop the unused armed return value with a blank assignment.
  - id: F5
    severity: NIT
    class: consistency
    origin: handler
    location: internal/gitexec/gitexec_unix_test.go:28
    summary: The comment atop TestRun_RemoteBounds lists the process-global state its rows set but omits the stall check seam the rows now install
    fix: In the doc comment of TestRun_RemoteBounds in internal/gitexec/gitexec_unix_test.go add the stall check seam to the list of process-global state that forces the rows to run serially.
---

### [MEDIUM] Production stall timer default is untested

**Location:** internal/gitexec/gitexec.go:55
**Issue:** Before the diff the silent row ran the real `time.AfterFunc(stallBound, check)` timer. All three stall rows now install `SetStallCheckForTest`, so nothing exercises the default: a wrong bound, a check that never runs or a stop that does not stop would pass. The board entry asks to remove the start-up race, not the coverage of the production timer.

### [LOW] Silent row readiness breaks when an EAGAIN start retry advances the attempt number

**Location:** internal/gitexec/gitexec_unix_test.go:219
**Issue:** `attempt` counts calls of the seam, and `runSpawning` calls `runAttempt` again after an EAGAIN failure (`gitexec.go:258-266`), after the seam was armed. The script then has run fewer times than `attempt`, `ready` never reports true, and the row ends at the 10 s deadline with a non-stalled error. EAGAIN is a host-pressure symptom, the condition this change targets. The retry attempt's `count >= attempt` is correct on the normal path.

### [LOW] Answering row does not prove its check ran

**Location:** internal/gitexec/gitexec_unix_test.go:283
**Issue:** The check fires only once the trace is non-empty. If it never does, the row ends at the deadline with every assertion satisfied, because `armed()` records arming, not running. The row's value, the size-0 condition sparing an answering remote, then goes unchecked. With the check running, dropping the size condition fails the row, so the design is sound when it runs.

### [NIT] Redundant armed() assertion in the unarmed row

**Location:** internal/gitexec/gitexec_unix_test.go:272
**Issue:** With an always-true `ready`, an armed check would fire at once and the row's existing `Stalled`, `runs` and kill assertions would fail, so the `armed()` assertion covers nothing they do not.

### [NIT] Serial-state comment omits the new seam

**Location:** internal/gitexec/gitexec_unix_test.go:28
**Issue:** The comment names the remote deadline, stall bound, kill reporter, pause gate and environment as the process-global state, but the rows now also replace the stall timer.

## Rejected

- Package-level `var` for `startStallTimer` against `PATTERN-test-speed` "Time is injectable" (handler's own candidate): the new variable has the same shape as `remoteDeadline`, `stallBound`, `runCommand` and `sleepBeforeRetry` in the same file, and the test is serial with its global state named.
- The other rows (`hung`, `RunWithin`, unarmed, answering) still race git start-up against the 1 s deadline (handler's own candidate): the board entry names the stall row, and the change description discloses the remaining deadline headroom.
- The stall row's 200 ms bound no longer proves a timing claim (focus question): the bound's value is only the `Timeout` the error carries, which the row asserts, and the three behaviors the row is meant to prove (stalled error, one retry, second stall ends the call) are all pinned.
- The commit message claims (focus question): each claim matches the diff, so no finding.
- Default `startStallTimer` changes behavior or the `tracePath` comment is false (focus question): the default is `time.AfterFunc(bound, check).Stop`, `defer` stands where it stood, and the body never reads `tracePath`.
