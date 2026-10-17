---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:103-111
    summary: D5 never says whether withdraw accepts a receiver creel no longer lists, yet a first withdraw that is refused makes lyx orch start fail
    fix: In D5's Fresh launch bullet, add that withdraw acts on the receiver by name whether or not creel still lists that strand, as release does, and answers an empty result when nothing of the caller's is queued there; add that lyx orch start runs the first withdraw before it removes the old strand. Add one acceptance criterion that a continue or adopt launched after lyx orch stop (orch strand removed, nothing queued) passes the first withdraw and starts the session.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:113-118
    summary: The bound after a failed second withdraw omits a failed release, which leaves ORCHWATCH's hold on the relaunched orch, and no acceptance criterion covers the failed-release case
    fix: In D5's Fresh launch bound, add a clause that while a release has failed and no retry succeeded, ORCHWATCH's hold stays on the relaunched orch, so creel reminds only ORCHWATCH's mail and no other sender's, until a retry succeeds or the hold lapses within the Hold bullet's one hour. Extend the acceptance criterion on a refused second withdraw to read "a refused second withdraw or a failed release" and add that the watcher renews no hold while idle with a failed release, so the hold lapses.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:115
    summary: The stated reason for the exposed inject, that its TTL counts only live unpaused time and does not delay the probe, does not match creel's TTL rule for an inject queued after the new session is live
    fix: In the D5 Fresh launch bound, replace "is typed on the relaunched session's first idle, unpaused probe, since its TTL counts only live, unpaused time and does not delay that probe" with "is typed on the relaunched session's first idle, unpaused probe if that probe comes within the ten-minute TTL, which runs while the session is busy; otherwise it expires". The exposure stays accepted; the clause only states the real upper bound.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:109
    summary: Only lyx orch start is said to clear the pause entry's resume-noticed mark when a withdraw removes a pause-resume line, but the watcher's retried withdraw and its phase-reset withdraw remove such lines too
    fix: In D5 (line 109), say that whichever caller's withdraw removes a pause-resume line clears the owning pause entry's resume-noticed mark under the record lock, the watcher's retry and phase-reset withdraws included, and that the orchengine creel seam's withdraw therefore returns the removed ids and subjects. Add the watcher cases to D9's sentence that a continued or adopted session gets the line posted again.
---

### [MEDIUM] D5 does not say whether withdraw accepts a receiver creel no longer lists

**Location:** `_lyx/discussion/decision-record.md:103-111`
**Issue:** The first withdraw runs "before it starts or binds the session", and a refusal of it makes `lyx orch start` fail and start nothing.
Creel's verbs refuse a receiver the hub does not list: `Store.strandTarget` in `internal/creelengine/verbs.go` answers `unknown strand`, and `Release` has an explicit carve-out for "a strand removed since" that the record gives `withdraw` nowhere.
`lyx orch start` removes the old strand (`RemoveStrand` in `internal/orchcli/start.go`, relaunch path) before it starts the new one, and `lyx orch stop` leaves no listed orch strand at all (D6 says as much for mail).
If the plan writer builds `withdraw` on `strandTarget` as `inject` is built, the first withdraw is refused in every continue, adopt and relaunch that follows a stop, so D5's own "no orch can be started" cost applies with no fault in the environment.
The orch strand name is fixed (`orchStrandName`), so the withdraw does reach the same receiver name across a relaunch, and the carve-out is the only open point.
The record leaves this choice to the plan writer, who cannot see the failure mode from D5.

### [LOW] The Fresh launch bound omits a failed release

**Location:** `_lyx/discussion/decision-record.md:113-118`
**Issue:** D5 records "the second withdraw or either release" as a failure the watcher retries, but the Bound names only the second withdraw's exposure.
A failed first release does not abort the start (only a failed first withdraw does), so the relaunched orch comes up under ORCHWATCH's old hold.
Under a hold creel reminds only the owner's mail, so the relaunched orch is reminded of ORCHWATCH's stale mail, a note request included, and of no other sender's mail, until a retry succeeds or the hold lapses.
The Hold bullet's one-hour lapse bound covers a dead watcher and does not mention this case.
The acceptance criterion for a relaunch with a refused second withdraw also leaves the release failure unpinned, and says "refused" where D5 covers an unreachable daemon too.
The exposure is small and time-limited, so this is a gap in the bound, not in the design.

### [LOW] The reason given for the exposed inject does not match the TTL rule

**Location:** `_lyx/discussion/decision-record.md:115`
**Issue:** Creel's TTL "runs only while the receiver is live and no pause covers it" and keeps running while the strand is busy or stalled (`internal/creelengine/doc.go`, Inject and interrupt).
An old inject queued between the two withdraws is queued after the new session is live, so the TTL runs through that session's first turn.
It is typed at the first idle probe only if that probe comes within ten minutes; otherwise it expires.
The clause "since its TTL counts only live, unpaused time and does not delay that probe" suggests the TTL cannot cut the exposure short, and it can.
The accepted exposure is an upper bound, so nothing in the design changes; the justification is wrong.

### [LOW] The resume-noticed mark is cleared only by `lyx orch start`

**Location:** `_lyx/discussion/decision-record.md:109`
**Issue:** The watcher withdraws too: it retries the second withdraw, and it withdraws ORCHWATCH's mail and injects when it resets a phase for another strand (D5 Hold bullet, `initCursor` in `internal/orchengine/watcher.go`).
Either withdraw can remove a pause-resume line that the old watcher posted and marked noticed, and D9 says a line withdrawn in the reset case is "posted again only to a continued or adopted session".
Nothing in D5 clears the mark in those two paths, so the line is never posted again.
The line is informational and the paths are rare, but D9 and D5 disagree about whether the repost happens.

## Rejected

- The D5 Fresh launch bound holds for what it states: injects the old watcher queued between the withdraws are bounded by the TTL rule above, and mail has no TTL, so the bound names "until a retry succeeds" for mail correctly; the strand name is the fixed `orchStrandName`, so the withdraw reaches the same receiver.
  The criteria for both withdraws succeeding and for a refused first withdraw match the Decision.
  The gaps are filed above as F2 and F3.
- The D5 "Steps that wait on the session" bullet counts a Stop whose stamp line and payload line fall in different `Runner.ReadEvents` reads.
  `ParseSessionSignals` (`internal/shuttleengine/claudeengine/signals.go`) returns the offset of the earliest line of a trailing run of stamp lines, so the decision's rule of resuming from that count leaves the stamp unconsumed and the next read sees both lines.
  Acceptance criterion "A step's Stop whose stamp line is read in one tick…" pins it.
- The D6 sender rule lets `NotifyPrime` tell the one retry-triggering refusal from the others: the refusal at `internal/creelengine/identity.go:56` is the only one that names a strand creel lists for a process in no pane, and the Scope entry for the typed refusal code, the exported client error value and the `creelengine` and `creelcli` docs lists the mechanism that replaces the plain `errors.New(resp.Error)` in `internal/creelengine/client.go`.
