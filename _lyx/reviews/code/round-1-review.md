---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/activity_test.go:289
    summary: The recovery branch where a waiterless running record turns terminal (activity.go noteWaiterlessRun Recover) has no test
    fix: In the subtest "a running record whose waiter is gone logs once per episode until it reads live", add a stretch that makes run-a waiterless again, reads, then rewrites run-a with outcome string(OutcomeDone) and pid 4343 and reads twice, and assert that the waiterlessRecovered Info count rises by exactly one. Keep it in the same subtest, not a new test.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/activity.go:85
    summary: A run directory whose record vanishes while its waiterless episode is open never logs the recovery Info and leaves its episode key in waiterlessRuns
    fix: In ReadAgentActivity, in the "if !found" branch, call waiterlessRuns.Recover(runDir, waiterlessRecovered, "runDir", runDir) before continue, so an absent record ends the episode like any other record that no longer reads running.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/activity.go:113
    summary: The recovery message says the run no longer reads running, which is wrong on the live-again path where the record still reads running
    fix: Reword the waiterlessRecovered constant to a neutral text such as shuttle run no longer has a gone waiting process, which fits both the live-again and the no-longer-running recoveries; the test reads the constant, so it needs no change.
---

### [LOW] Terminal-outcome recovery is untested

**Location:** internal/shuttleengine/activity_test.go:289
**Issue:** The decision record and `doc.go` say a record that no longer reads `running` ends the episode with one Info.
The new subtest ends the episode only by the record reading live, so deleting the `Recover` at `activity.go:124` leaves the suite green.

### [LOW] Absent record leaves the episode open

**Location:** internal/shuttleengine/activity.go:85
**Issue:** The `!found` branch continues before any `waiterlessRuns.Recover`.
A run directory that logged the Warn and then lost its record gets no pairing Info, and its key stays in the map for the process lifetime.

### [NIT] Recovery message misdescribes the live-again path

**Location:** internal/shuttleengine/activity.go:113
**Issue:** "no longer reads running with its waiting process gone" is inaccurate when the record still reads running and a new waiter is alive.

## Rejected

- Holistic check of focus questions 1 to 4 (Warn then Debug then one Info, no-pid and finished records silent, live-again recovery exactly once, unchanged liveness, notice text and record shape): all held, so no finding.
- Lens code-affected-set and code-pattern-fit returned no findings; the holistic read of docs, `PATTERN-log-levels` and the commit messages agrees.
- The code-correctness lens's doubt about the test's dead pid 4343: rejected, because `activity_test.go:80-83` stubs `isAlive` to accept only 4242.
