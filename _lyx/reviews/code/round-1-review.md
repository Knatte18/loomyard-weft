---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shuttleengine/wait.go:330
    summary: >-
      The board entry also asks that a turn answering a prompt the operator typed raise no held notice, and the diff withholds the notice only for a turn end with background work outstanding; the omission is the writer's own claim and no Added after Discussion entry records it
    fix: >-
      Leave the code in place and defer this finding to the run's parent, since recording or refusing the omission with `lyx loom decision add` is the parent's and lies outside the fixer's write surface.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/smoke_helpers_test.go:99
    summary: >-
      providerlessShuttleConfig rewrites only startup_timeout_s and keeps the template's start_retries of 1, so each providerless fixture start that ends died with its strand removed is now launched a second time after a 5 s wait, doubling the cost the helper's header says it bounds
    fix: >-
      In providerlessShuttleConfig, also replace the template line `start_retries: 1` with `start_retries: 0` (strings.Replace on the template text, as the existing startup_timeout_s rewrite does), and extend the comment above it to say the fixtures take one launch only.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:439
    summary: >-
      The new TestStartup_DiedStartLaunchesAgain and TestStartupCaptureExcerpt were inserted between the keyTimesReed doc comment and its type declaration, so the type lost its doc comment and the test carries a stray first line about keyTimesReed
    fix: >-
      Move the line `// keyTimesReed records the clock time of every SendKey call and fails the calls whose key is failKey, after recording them.` from above TestStartup_DiedStartLaunchesAgain to directly above `type keyTimesReed struct` (startup_test.go:536), leaving the test's own doc comment starting at its TestStartup_DiedStartLaunchesAgain line.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:442
    summary: >-
      TestStartup_DiedStartLaunchesAgain has no row for a timeout start, so dropping the OutcomeDied check in startRetryable fails no test, and it asserts neither that the returned error is the last attempt's nor the 5 s start wait and the 1 s then 2 s removal waits on the run clock
    fix: >-
      Add a row to TestStartup_DiedStartLaunchesAgain whose start ends timeout with the strand removed and expects one launch; assert in the retry-on row that the error of the final failed attempt names the second run directory when both attempts fail; and record the fake clock's sleeps in that test and in the removal-retry rows of TestStartup_PaneNotLiveMidStartup, asserting startRetryDelay and then 1 s and 2 s.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/overlay_test.go:1
    summary: >-
      TestTmuxExecRetriesEAGAIN has no row that sends non-nil stdin through runStdin and checks the retry is handed the same bytes
    fix: >-
      Add a runStdin row to TestTmuxExecRetriesEAGAIN whose execHook records the stdin of each call, fails the first with an EAGAIN-wrapping error and succeeds the second, and asserts both calls saw the same non-nil bytes.
---

### [MEDIUM] Operator-typed prompt turn left out of the held-notice rule

**Location:** `internal/shuttleengine/wait.go:330`, `internal/shuttleengine/hold.go:107`
**Issue:** The board entry's third bullet says a turn that answers a prompt the operator typed in the session's pane is not reported as held, beside the background-work case.
The diff covers only the background-work case.
The decision record and `_lyx/landing/summary.md` explain the omission, but the record carries no `Added after Discussion` entry for it, so it stands as an unrecorded deviation from the entry, deferred to the run's parent.
The explanation itself is sound against the code: shuttle holds no record that tells an operator's typed text from a parent's answer.

### [MEDIUM] Smoke fixtures now launch a failing start twice

**Location:** `internal/loomcli/smoke_helpers_test.go:99`, `internal/shuttleengine/template.yaml:6`
**Issue:** The providerless fixtures point the provider at a path that cannot exist, so each start ends died with the strand removed, which `start` now retries after five seconds.
The helper shortens only `startup_timeout_s`, so every such start costs a second startup window plus the wait, against a suite whose header says it exists to stay bounded.

### [LOW] Doc comment of `keyTimesReed` displaced by new tests

**Location:** `internal/shuttleengine/startup_test.go:439`
**Issue:** The first line of the new test's doc comment is the `keyTimesReed` type's comment, and the type at line 536 has none.

### [LOW] Start-retry bounds without a pinning assertion

**Location:** `internal/shuttleengine/startup_test.go:442`
**Issue:** The record bounds the retry to a died start with a removed strand and returns the last attempt's error; the test's rows cover retry on, retry off and a failed removal only.
A timeout start, the last-attempt error and the run-clock waits are stated bounds no assertion pins.

### [LOW] Stdin replay on a retry not asserted

**Location:** `internal/reedengine/overlay_test.go:1`
**Issue:** `runStdin` rebuilds its command and reader per attempt, which the code does correctly, but no row of `TestTmuxExecRetriesEAGAIN` sends stdin through a retry.

## Rejected

- lens:code-tests, no row for existing output files: the three negative startup verdicts that reach `abandonStartup` consult `allOutputFilesExist` first (`wait.go:682`, `wait.go:722`), so `startRetryable` needs no such guard and a row would pin pre-existing behavior.
- lens:code-correctness F1, removal retry after a partly done removal: unconfirmed, and a single attempt failed the same way before the change.
- lens:code-correctness F2, no `allOutputFilesExist` in the start loop: same reason as the first item.
- lens:code-affected-set F2, `docs/overview.md` shuttle row: `doc.go` is the module doc and was updated, and the row is a summary that never listed per-key behavior.
- Board entry's "never classify an unverifiable liveness check as died": the diff retries EAGAIN before it can surface, and the record decides the abandoned attempt still records `died`; the outcome set is closed, so no change to the diff is called for.
