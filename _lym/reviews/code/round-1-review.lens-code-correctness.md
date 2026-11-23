```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

1. `haltOnEndedSession` with `keepsSession=false` (the `ReviewEnded`-at-entry wait or the merged-at-entry wait): returns nil before reading liveness. Correct.
2. `haltOnEndedSession` with `rec.ReviewEnded=true` set mid-wait, after `endReview` or a `resumeSession` that skipped the restart: returns nil. Correct.
3. `haltOnEndedSession` with `keepsSession=true` and an empty `StrandGUID`: not reachable. `keepsSession` is true only after `ensureSession` succeeded, and on success it records the GUID (`prsession.go:112-126`). Its failure returns an error before `wait`. `settleSendBack` clears the GUID only at call entry. Correct.
4. `haltOnEndedSession` with the strand live and Retiring: `sessionLiveness` reads it as not live (`landingdeps.go:1064`), so the row halts. Correct, since a retiring strand is going away.
5. `haltOnEndedSession` with the strand dead: `halt` returns Stuck and Halt. `endSession`'s `stopLiveSession` then reads it not live and stops nothing. The resume re-enters `openPullRequest`, and `ensureSession` resumes with `rec.SessionID` (`prsession.go:79-101`). So "lym loom resume … resumes the recorded session" holds.
6. `haltOnEndedSession` with a cancelled context and a dead strand: `halt`, through `stuck`, returns the cancel error, so the row records no false halt. Correct.
7. `haltOnEndedSession` when `SessionLive` returns an error: the call ends with a plain error. Not correct as a design choice; see finding 1.
8. `wake` on the `!polled` path, with the session dead and nothing else due: halts at `:706`. Correct.
9. `wake` on the `!polled` path because of a transient poll failure, on the wake that first sees `Continue`: `continueSeen` is updated at `:703` before the check. A session that exited after `pr-continue` halts instead of ending cleanly. See finding 2.
10. `wake`, polled, with `judge` returning nil, `stopped=false` and the session dead: halts at `:735`. Correct.
11. `wake`, polled, with `stopped=true` (the session stopped around the read, then `resumeSession` ran): the check is skipped. `resumeSession` either made the session live or saw `ReviewEnded`. Correct.
12. `wake` where `sessionStopWanted` holds but the session was already dead, so `stopLiveSession` returns false: the check runs and halts. Correct.
13. `wake` with a pause covering the run: `holdWhileCovered` stops the session and resumes it before the check, so the row's own stop is never misread as a death. Correct.
14. `wake` with merged, approved, ended or send-back: these return before the check. Correct.
15. `openPullRequest` with `ReviewEnded` at the gated head: `keepsSession=false`. Correct.
16. `openPullRequest` with a closed PR: `ensureSession` runs, so `keepsSession=true`. Correct.
17. `openPullRequest` with a merged PR: no session, `keepsSession=false`. Correct.
18. `openPullRequest` on the open or new path: `ensureSession` runs, then `wait(..., true)`. Correct.
19. Line `:735` under the mutation (`end == nil`): it would drop the stay-open mail on the live path and swallow the halt on the dead path. The production code is correct. Test coverage of this branch is the code-tests lens's subject.

**Findings**

1. **MEDIUM, design.** At `internal/landingshed/prreview.go:752-755`, one transient reed read failure on any wake ends the review call as a producer error and stops its live session.
   - `haltOnEndedSession` now calls `SessionLive` on every wake that leaves the review open. That is up to once a second while the head moves, and at least once a minute otherwise.
   - `SessionLive` goes through reed's `Status`. `Status` takes reed's op write lock, checks the tmux session and lists panes (`internal/reedengine/lifecycle.go:1297-1312`, `internal/reedengine/lock.go:138-157`).
   - Any failure there returns `&ending{err}`. `Call` then runs `endSession`, which stops the live session because the error is no cancellation (`prsession.go:295-301`). The driver hands the producer error to the tackler (`internal/sheddriver/doc.go:45-46`).
   - Before this diff the wait read reed only on a wake whose read could end the review. The wait's other periodic reads judge a transient failure with warn-once-and-continue (`pollFailure`, `prreview.go:915-927`).
   - Fix: treat a failed liveness read like a transient poll failure: Warn once per episode and keep waiting. Keep the error path for the entry read in `ensureSession`.

2. **LOW, design.** At `internal/landingshed/prreview.go:703-706` (with `:898`), a transient status-poll failure on the wake that first sees the continue flag can halt a review that should have ended cleanly.
   - Case: the agent runs `lym loom pr-continue`, then exits.
   - The forced poll fails transiently, so `polled=false`. `state.continueSeen` is already set to true at `:703` before `haltOnEndedSession` runs at `:706`, and that check halts on the dead session.
   - The next wake would no longer force a poll, since `rec.Continue && !state.continueSeen` is now false, so the halt wins.
   - `summary.md` claims "an agent that runs `lym loom pr-continue` and then exits still ends the review cleanly". That holds only when the forced poll succeeds. The parent must resume a review the operator or the parent already ended.
   - Fix: on a wake where the poll was forced by a newly seen continue flag but did not read a status, skip the liveness check. Alternatively, update `continueSeen` only after a successful poll.

Tool calls:
- grep of `internal/sheddriver` for re-step policy and of reed's `Status` body: the context lacked how the driver treats a producer error and what `SessionLive`'s underlying read can fail on.
- read of `internal/sheddriver/doc.go:30-50`, reed's `withOpLock` and `requireSessionLocked`: the context lacked the lock and tmux failure modes behind `SessionLive` and the driver's handling of a plain producer error.
