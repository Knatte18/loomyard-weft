---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:706
    summary: The liveness check on a wake that read no status halts before the row reads a status that may already end the review, such as a merge, an approval, or a continue flag whose forced poll failed transiently.
    fix: "In `wake`, change the `!polled` branch back to `return nil`, so `haltOnEndedSession` runs only at the `else if` after `judge` on a wake that read the status. Change the `wake` doc line to \"A wake that reads the status and leaves the review open checks that the session the call keeps is still live.\" In `_lym/landing/summary.md`, under \"Where the check runs\", drop \"on a wake that read no status and\" so the bullet says the check runs on a wake whose judgement left the review open."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:752
    summary: A failed liveness read on any open-review wake ends the call as an error, and `endSession` then stops the live session, where the wait's other periodic reads warn once and keep waiting.
    fix: "Add a `liveWarned bool` field to `prWaitState`. In `haltOnEndedSession`, when `SessionLive` returns an error, log `logger.Warn(\"landingshed: could not read whether the PR-Review session is live, the wait goes on\", \"row\", p.name, \"cause\", err)` only when `state.liveWarned` is false, set it true, and return nil. Clear `state.liveWarned` after a read that succeeds. Leave the error path of `ensureSession` unchanged."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go:1001
    summary: "No test pins the halt on a wake that read the status (`prreview.go:735`, the mutation survivor). The dead-session row halts on a wake that read none, yet once the wait idles, every wake polls and line 735 is the only halt."
    fix: "In the `ASessionThatEndsWhileTheReviewIsOpenHaltsNamingTheResume` row, change `scripts` to `statusScript(open, open)`. In its `onSleep` at `n == 1`, also advance `w.now` by `prPollInterval`, so wake 2 polls, judges the open review, and halts at the `else if` after `judge`. Keep the row's `wantHalt`, `wantReason` and session-count assertions."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go:1001
    summary: "No row reaches a failed liveness read inside the wait. `ASessionLiveErrorIsAnError` fails in `ensureSession` at entry, before the wait runs."
    fix: "Add a `TestPRReview_Wait` row with `record: seed(\"h1\")` and `scripts: statusScript(open)`. Its setup sets `sessionsOf(w.deps).liveErr` in `onSleep` at `n == 1` and cancels at `n >= 3`. It asserts the behavior F2 settles: `wantErr: context.Canceled`, one Warn naming the liveness read in the captured logs, and one start and no stop."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/landingshed/prreview.go:749
    summary: "`AReviewEndedAtEntryKeepsNoSessionSoItsDeadStrandDoesNotHaltTheWait` does not pin the `keepsSession=false` its name claims, because the `rec.ReviewEnded` guard returns nil first and the row passes with either flag value."
    fix: "Change the guard in `haltOnEndedSession` to `if !state.keepsSession {`, dropping `|| rec.ReviewEnded`. `endReview` is the only writer of `ReviewEnded`, and it ends the call in the same wake, so a wait entered on an ended review is the one case, and it passes false."
---

### [MEDIUM] The no-status liveness check halts before the review's ending is read

**Location:** `internal/landingshed/prreview.go:706`

**Issue:** `wake` runs `haltOnEndedSession` on a wake that read no status, so the halt wins over a status the row has not read yet.
The case the lens traced: the agent runs `lym loom pr-continue` and exits.
The wake that first sees the flag forces a poll, and the poll fails transiently, so `polled` is false.
`continueSeen` is already set at line 703, so no later wake forces the poll, and the check at line 706 halts on the dead session.
The same holds when the operator merges or approves the pull request on GitHub and closes the session pane before the next minute's poll.
The resume recovers in each case, but the parent must act on a review that has already ended.
`_lym/landing/summary.md` claims that an agent that runs `lym loom pr-continue` and then exits "still ends the review cleanly", which holds only when the forced poll succeeds.
Running the check only after `judge` delays detection by at most one poll interval, a minute, against the issue's six-minute hang.

### [MEDIUM] A transient reed failure ends the review and stops its session

**Location:** `internal/landingshed/prreview.go:752`

**Issue:** `haltOnEndedSession` reads `SessionLive` on every wake that leaves the review open, which goes through reed's `Status` under its op lock.
A failed read returns `&ending{err}`, and `Call`'s `endSession` stops the live session, since the error is no cancellation.
Before the diff, the wait read reed only on wakes whose read could end the review.
The wait's other periodic reads, the status poll and the title edit, warn once per episode and keep waiting (`pollFailure`).

### [MEDIUM] The halt after `judge` is untested

**Location:** `internal/landingshed/prreview_wait_test.go:1001`

**Issue:** The mutation report's survivor at `prreview.go:735` negates the `else if end := p.haltOnEndedSession(...)` condition, and every test passes.
The dead-session row halts at line 706: wake 2 falls within a minute of the entry read, so it reads no status.
Once the gap between wakes reaches `prWaitCeiling`, which equals `prPollInterval`, every wake polls and line 735 is the only halt.
With F1 applied it is the only halt at all.

### [LOW] A failed liveness read inside the wait is untested

**Location:** `internal/landingshed/prreview_wait_test.go:1001`

**Issue:** `prreview_session_test.go`'s `ASessionLiveErrorIsAnError` sets `liveErr` before the call, so the read fails in `ensureSession` at entry.
No row fails the read in `haltOnEndedSession`, and the two sites wrap the error with the same message.

### [LOW] The ended-at-entry row does not pin the flag its name claims

**Location:** `internal/landingshed/prreview.go:749`

**Issue:** The row's record carries `ReviewEnded`, so `haltOnEndedSession` returns nil on `rec.ReviewEnded` whatever `keepsSession` holds.
Passing true at `prreview.go:306` keeps the row green.
The guard adds nothing to the flag: `ReviewEnded` is written only by `endReview`, whose wake ends the call.

## Rejected

- code-scope and code-affected-set reported no findings. I confirmed: the diff builds the board entry's halt option, the halt reason carries a `way forward:` clause, and the `Root-Cause:` trailer names the defect the diff removes.
- The tests lens says the merged-at-entry `keepsSession=false` path is unobservable. That is not quite so: a pause at the first wake followed by a transient poll failure reaches the no-status check with a dead strand. The flag guards that path, so it stays, and no finding is filed.
