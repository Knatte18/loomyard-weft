---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: D5's revive-versus-pause bound rests on a false premise; a landed line need not read busy at the pause's first poll, so an idle-stalled hub lets the pause settle at once and the revived turn runs unpaused and unrecorded
    fix: In D5, make revive keep the pause record's write lock after the conditioned send lands until the session's hook-derived state reads a turn start past the send (shuttle's turn_start signal past the pre-send events offset), bounded by a stated wait; a send that lands with no turn start seen within that wait is reported as such. Rewrite line 81 to state that guarantee, and state the remaining case plainly - a revived turn that ends before the pause's poll (for example a repeated API error) is not recorded, so resume gives it no line and a later revive does. Name the turn-start wait in Scope In's conditioned-send bullet and in the shuttleengine doc.go constraint, and keep the Acceptance criterion at line 167 matching the rewritten bound.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:75
    summary: The conditioned send's refuse-at-once rule covers only the session state and cause; a draft in the box or operator activity on an idle-stalled api-error session still falls into today's idle wait of up to send_ready_timeout_s, held under the pause record lock, so the at-most-one-send bound on a concurrent pause is unquantified
    fix: In D5, decide whether the conditioned send also refuses at once on every other hold its readiness reading names (a draft, operator activity, an unmatched turn start), and say so on line 75. Then restate the Bound on line 82 with the longest time revive holds the pause record lock per send (the submit window if it refuses every hold; send_ready_timeout_s plus the submit window if it waits), since pauserecord.Update blocks on that lock with no timeout and pause, resume and the orch watcher's record write all go through it.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:95
    summary: D6 and its Acceptance criterion say a call cancelled during the hold ends with the cancellation at release, but when the caller's WithTimeout deadline passes during the hold first, the context already reads DeadlineExceeded and a later cancel is unobservable, so by D6's own rule the call runs on to the moved deadline
    fix: On line 95, qualify the claim - a call whose context is cancelled during the hold, before its own deadline passes, ends with the cancellation at release; once the deadline has passed during the hold, a later cancellation is not observable and only the moved deadline ends the call. Apply the same qualifier to the first half of the Acceptance criterion on line 159 (cancelled during the hold before its original deadline passes).
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:115
    summary: D8 says no in-memory pause handling is lost, but the watcher's handled set is memory only; an entry the old image settled during its grace because the orch read idle is not settled for the new image, which records and interrupts the orch at that grace end if it then reads busy
    fix: In D8, either re-exec only at a tick with no live hub entry at all, or replace the clause on line 115 with the actual effect - an entry the old image settled during its grace is handled afresh by the new image, which interrupts the orch at the grace end only if it reads busy then - and add it to D8's Bound.
---

### [BLOCKING] D5's revive-versus-pause bound does not close the window

**Location:** `_lyx/discussion/decision-record.md:81`, with the Acceptance criterion at line 167

**Issue:** D5 claims a pause entry added after revive's coverage read is "added only after the line has landed, when the pause's grace poll reads the session busy and its interrupt pass records it".
Two facts in the tree break that.
First, a landed send is not a started turn: `confirmSubmitted` (`internal/shuttleengine/send.go`) accepts a landing when the input box no longer holds the sent text *or* a turn start appears past `sentAt`, so the send can return before the hook-written turn start reaches the events file.
Second, the pause reads busy from that hook-derived state only (`shuttlePair.Busy` in `internal/pausecli/deps.go` via `readTurn`), and `pauseengine.Pause` breaks its grace poll on the first reading where no covered session is busy and no network operation is in flight.
In the case revive exists for — a network outage, every covered session idle-stalled — the pause's first poll right after revive releases the lock reads every session idle, including the just-revived one whose turn start is not yet written, ends the poll at once, finds no running turn in `interruptSession`, lists the session as `already_idle` and records nothing.
The revived turn then starts under the live entry, is never interrupted, and `resume` gives it no line.
The lock narrows the window to the gap between the box clearing and the turn start landing in the events file; it does not close it, so the either/or on line 81 and the criterion "A pause added while a `revive` line is in flight interrupts and records that session" are not achievable as written.
A separate, benign residue the same sentence omits: a revived turn that ends before the pause's poll (the API error recurring during the outage) reads idle and is not recorded either, which is correct for the pause but means `resume` gives it no line.

### [MEDIUM] The conditioned send's lock-hold time is unbounded in practice

**Location:** `_lyx/discussion/decision-record.md:75`, with the Bound on line 82

**Issue:** The conditioned send "refuses at once, with no idle wait, when the reading shows anything else; otherwise it is today's guarded send".
Today's guarded send (`awaitIdleSession`, `busyReading`) holds on a draft in the box, an unmatched turn start or operator activity in the pane, each of which can coexist with a state that reads idle-stalled with cause `api-error`, and waits up to `send_ready_timeout_s` (60 s by default) for it, then up to the submit window.
Whether those holds count as "anything else" is left open, and revive holds the pause record's write lock across that whole wait.
`pauserecord.Update` takes that lock through `state.UpdateJSON` → `lock.AcquireWriteLock`, which blocks with no timeout, so a concurrent `pause` — the operator's stop — `resume` and the orch watcher's record write can each stall for the full wait.
"At most one send" is a bound in count only; the rubric's attack-surface rule asks what bounds the new lock hold, and the Decision does not state a duration.

### [MEDIUM] D6's cancellation-during-hold claim fails once the deadline passes during the hold

**Location:** `_lyx/discussion/decision-record.md:95`, with the Acceptance criterion at line 159

**Issue:** D6 states "once that deadline has passed, only the moved deadline ends the call" and then, unqualified, "a call cancelled during the hold ends with the cancellation at release".
For a caller context built with `context.WithTimeout` whose hold outlasts that timeout, the context is done with `DeadlineExceeded` before any cancellation; a later `cancel()` changes neither `Done` nor `Err`, so at release the transport cannot see the cancellation and, by D6's own first rule, runs the call under the moved deadline.
The two sentences disagree for that case, and the criterion's first half ("cancelled during the hold ends with the cancellation once the hold releases") cannot be met for it.
The rest of D6's cancellation claims hold: a cancellation before the original deadline reads `Canceled` and ends the call, during the hold or after release, and the post-release half of the criterion already carries the "before its original deadline passes" qualifier.

### [LOW] D8's "no in-memory pause handling is lost" is not true of the handled set

**Location:** `_lyx/discussion/decision-record.md:115`

**Issue:** `handleHubEntry` (`internal/orchengine/pause.go`) marks a hub entry handled during its grace when the orch reads idle and has no network operation in flight, and that mark lives only in `pauseHandled`.
D8 lets the watcher re-exec while such an entry is live, since it counts as handled.
The new image's `settledWithoutInterrupt` settles it only if the caller is the orch, the orch is recorded, the orch launched after it, or its grace end precedes the new first tick; none applies to an entry whose grace end is still ahead, so the new image records and interrupts the orch at that grace end if it reads busy then, which the old image would not have done.
The D8 Bound's wording about a live entry from the new image's first tick does match the code: the cycle machine is held from the first tick, and `settledWithoutInterrupt` treats an entry whose grace end precedes that tick as settled, so only an entry whose grace end is still ahead can bring an interrupt — not a defect.

## Against prior rounds

Read after this review was saved.
Neither prior fixer disputed or deferred anything, and round 2's fixes are in the text as reported.
F1 here concerns the mechanism round-2 F1 itself proposed: the lock closes the gap before the send, not the gap between a landing and its turn start.
F2 here is the duration side of round-2 F2's bound, which stated a count only.
F3 here is the during-the-hold half of round-2 F3, which round 2 judged not a defect because it read the hold as ending at release with the cancellation; that holds only when the cancellation precedes the deadline.
F4 here departs from round 2's note that no started handling is lost: that is true of `pauseFirstEsc`, but an entry already in `pauseHandled` is handled afresh by the new image.

## Focus verdicts

- D5, revive versus a hub pause: the bound does not hold when covered sessions are idle-stalled; the lock narrows the window and does not close it (F1).
- D5 and Scope In, shuttleengine behaviors: today's `Runner.Send` → `sendVerified` → `awaitIdleSession` has no state-and-cause precondition and always runs the idle wait, so the conditioned send is new; Scope In's second bullet names it and the docs constraint names `shuttleengine`'s `doc.go`, so the change itself is named. Its bound is incomplete (F2).
- D6, cancellation under a `WithTimeout` context outlasted by the hold: the claims hold except the unqualified cancelled-during-hold sentence and criterion (F3).
- D8, Bound wording against `settledWithoutInterrupt`: matches; the adjacent claim about in-memory handling does not (F4).
