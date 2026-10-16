---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5, paragraph beginning "Shuttle's engine send does not consult the pause record"
    summary: D5's stated bound on the revive-versus-pause residual window is false; a pause whose grace ends at once on a hub of idle-stalled sessions records none of them, so a revive line landing after that pass runs a session under a live pause, unrecorded
    fix: In D5, replace the sentence that says the pause's own grace-then-interrupt bounds the residual window with a mechanism that closes it, and state that as the bound. Recommended - revive reads a session's coverage and sends its line while holding the pause record lock (the lock pause takes to add its entry), so a pause entry is either live at the read, and the session is skipped, or added only after the line has landed, when the pause's grace poll reads the session busy and its interrupt pass records it for resume's line. State the cost in the bound - the lock is held for one guarded send at a time, so a concurrent pause waits at most one send. Keep the Acceptance criterion on a pause live at the coverage read, and add one - a pause added while a revive line is in flight interrupts and records that session.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5, the bullet beginning "every other session gets one fixed line"; Scope In list
    summary: D5 describes shuttle's guarded send as refusing with no idle wait unless the state reads idle-stalled with cause api-error, but today's send has no state precondition and waits up to the send-ready window for any idle state; the shuttleengine change this needs is in neither Scope nor a Decision
    fix: In D5, say that shuttleengine gains a conditioned send - a send whose ready reading must name a given session state and cause, which refuses at once with no idle wait when it does not - and that revive uses it. State its bound in that Decision - it can only refuse more than today's send, never type where today's would not. Add shuttleengine's conditioned send to Scope In, and add shuttleengine's doc.go to the Docs-in-the-same-commit constraint.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, sentence beginning "When a call was held"; Acceptance criteria, the held-call cancellation criterion
    summary: Once a held call's original deadline has passed, its context is already done with a deadline error, so a later cancellation of the caller or an ancestor context is unobservable to the transport; D6's claim that every non-deadline cancellation still ends the call, and the criterion that a call cancelled after release ends at once, fail in exactly the held-past-deadline case #606 is about
    fix: In D6, narrow the sentence to - every cancellation of the caller's context that arrives before its own deadline passes still ends the call; once that deadline has passed, only the moved deadline ends the call, so the extension is the longest a late cancellation can be ignored. Amend the Acceptance criterion to - one cancelled after release, before its original deadline passes, ends at once.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D8, sentence beginning "a hub pause or a revive in the gap"
    summary: D8 says the new image handles a live hub entry from its first tick, but the watcher settles with no interrupt any entry whose grace end precedes its first tick, so an entry whose grace ends inside the re-exec gap leaves a busy orch running under the pause
    fix: In D8's Bound, replace "the new image handles a live hub entry from its first tick" with - the new image holds the cycle machine under a live hub entry from its first tick, and interrupts a busy orch only for an entry whose grace end is still ahead; an entry whose grace ended in the gap leaves the orch uninterrupted, as with no watcher, which the pause's no-live-watcher report already says.
---

### [BLOCKING] Revive's residual window against a pause is not bounded by the pause

**Location:** `_lyx/discussion/decision-record.md` D5, paragraph beginning "Shuttle's engine send does not consult the pause record".

**Issue:** D5 says a pause entry added after revive's coverage read and before the line lands is bounded by "the pause's own grace-then-interrupt", which records the session so `resume` gives it its line.
`internal/pauseengine/doc.go` says the pause polls each second until every covered session reads idle, then records and interrupts only each session that reads a running turn.
An idle-stalled session reads idle (`pausecli`'s `Busy` derives from the running-turn reading), so in the outage revive exists for, where the covered sessions are stalled, the grace ends at the first poll and the interrupt pass records nothing.
A revive line that lands after that pass, which the paste-and-verify of a send easily outlasts, starts a turn under a live pause, unrecorded, and nothing stops it.
That is the guard-bypass the rubric's attack-surface rule asks to be bounded, and the stated bound does not hold.
The focus question's answer: the state bound is enforced at the send's own ready reading for task sessions (given F2) and at delivery for the orch, but the pause bound is enforced only at revive's earlier coverage read.

### [MEDIUM] D5 presents a new shuttle send mode as existing behavior

**Location:** `_lyx/discussion/decision-record.md` D5, the bullet beginning "every other session gets one fixed line"; Scope In.

**Issue:** `Runner.Send` in `internal/shuttleengine/run.go` runs `sendVerified`, whose `awaitIdleSession` (`send.go`) waits up to `SendReadyTimeoutS` (default 60 s) for a ready reading, and that reading is ready on idle-done, idle-stalled of any cause and idle-fresh (`shuttleengine/doc.go`).
No send takes a state precondition or skips the idle wait.
D5's "That send refuses, with no idle wait, unless the session's state reads idle-stalled with cause `api-error`" therefore describes a shuttleengine change, not shuttle's guarded send as it stands.
Scope In lists only the tender module and the GitHub, watcher and attach changes, so a plan writer reading D5 as written would wire revive to today's `Runner.Send`, which waits out a busy session and then types into it, the case D5's "never waited for" rules out.

### [MEDIUM] Cancellation after the original deadline is unobservable to the transport

**Location:** `_lyx/discussion/decision-record.md` D6, sentence beginning "When a call was held"; the Acceptance criterion on cancellation.

**Issue:** The callers derive their deadline with `context.WithTimeout` (`landingshed`'s `publishGitHubTimeout` sites, `selfreportengine`'s `callTimeout`, `fabriccli`'s `openPRListTimeout`).
A call held past that deadline reaches release with `req.Context()` already done and `Err()` reporting the deadline.
A later explicit `cancel()` leaves that error unchanged, and an ancestor's later cancellation does not propagate into a context that is already done, so the transport, which sees only the request's context, cannot detect either.
D6's "every cancellation of the caller's context other than its deadline still ending the call" and the criterion "one cancelled after release ends at once" are then unachievable whenever the hold outlasted the deadline, which is the #606 case.
The consequence is small, since the moved deadline still ends the call, so the defect is the false claim and the criterion that would test it.

### [LOW] D8 overstates what the new watcher does with an entry whose grace ended in the gap

**Location:** `_lyx/discussion/decision-record.md` D8, Bound.

**Issue:** `settledWithoutInterrupt` in `internal/orchengine/pause.go` treats an entry as settled when `entry.GraceEnd.Before(w.firstTick)`, so the new image neither interrupts nor records a busy orch for a pause whose grace ended during the re-exec gap.
"The new image handles a live hub entry from its first tick" reads as including that interrupt.
The hold of the cycle machine does apply from the first tick, and the pause's report already says no watcher is live, so only D8's wording is off.
The rest of the D8 focus question checks out: `pauseHandled` and `pauseFirstEsc` are the only in-memory pause state, an entry still awaiting its second key is not yet in `pauseHandled`, and D8's re-exec condition waits for both, so no started handling is lost.

## Against round 1

Read after this review was saved.
Round 1's fixer disputed and deferred nothing, and none of its fixes regressed.
F1 and F4 here concern text round 1 itself proposed: round-1 F2's "the pause's own grace-then-interrupt bounds it" and round-1 F6's "the new image handles a live hub entry from its first tick".
F2 here is the scope left behind by round-1 F1's fix, which gave D5 a send behavior that shuttle does not have without adding the shuttle change to Scope.

## Focus verdicts without a finding

- D5 and D3 session scope: not a defect.
  `pausecli`'s `hubDeps` excludes the prime from the pairs, and D5 adds the orch from the prime's orch state for the hub scope only, while D3 lists every prime session read-only; the two Decisions say this consistently, and the Acceptance criteria pin the orch-only reach.
- D6 wall-clock budgets above the transport: not a defect.
  The rule names a mechanism a context deadline can follow, one fresh deadline per request in the caller's own loop, which `fabriccli`'s paginated open-PR listing (the one several-request caller under one deadline) can adopt, and it agrees with the Bound, which describes the transport's per-request property that the rule then makes the only case.
  No shed, loom or batten step wraps these calls in an outer context deadline today.
- D6 cancellation during the hold: not a defect.
  `pauserecord.Network` waits on `context.Background()`, D6 keeps it blind on purpose, and both D6 and its criterion say the call ends at release; F3 concerns cancellation after release only.
