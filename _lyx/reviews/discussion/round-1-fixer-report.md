# Round 1 fixer report

Every finding is fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.
F12 withdrew the old D2, so the old D3–D8 are D2–D7 in the record now; the findings below use the old numbering, as the review does.

## Changed

- **F1 (D3, BLOCKING):** the same-mode request rule no longer compares with `PhaseEnteredAt`.
  `State` records `CycleStartedAt` at entry into the handoff-requested phase.
  A compact request is removed when the compaction completes if it was made at or after the cycle's start and not after the boundary; a clear request is removed when `/clear` is confirmed, if made at or after the start.
  Acceptance criteria cover the handoff-phase request, the post-boundary request and the clear cycle; scope names the new `State` field.
- **F2 (D7, BLOCKING):** the watched fact moves out of the seed into a `batten-watched` marker holding batten's pid, in the run's scratch directory, its path declared in `shedrun`; the driver render counts the run as watched only while the pid is live.
  Scope and Constraints name `shedrun` and `PATTERN-lyxdirs-single-declarer` / `PATTERN-durable-vs-ephemeral-state`; the criterion states the seed is unchanged.
- **F3 (D7 and D5, BLOCKING):** a notice is recorded as sent only when `QueueNotice` reported it queued, so a failed or unqueued notice is retried; a restarted batten sends the notices not yet sent.
  The batten that dies after the render is a new Open risk and a stated Bound of D6.
  Two criteria added.
- **F4 (D8, BLOCKING):** the window is created detached and never selected, and reed addresses its strand window by id instead of the session's current window.
  Scope and the docs list name the reed change, a criterion covers a reed op while the batten window is current, and `reed-multi-window` stays out of scope for strands in other windows.
- **F5 (D8):** the op sets `remain-on-exit off` for its window; a dead pane under the name is replaced; the run lock, not the window check, is named as what stops a second batten; the window is addressed by id and the name is for display.
- **F6 (D5):** a report counts only when its file time (the park marker's, for a parked stop) is at or after `since`; `since` is defined as the newest history entry's time, a lower bound on the state's age.
  Criterion added.
- **F7 (D5 and D4):** the pending notice is sent at the latest before the call returns for any reason, and the done notice before `Done`.
  Stated in D3 and in the criterion for the done row.
- **F8 (D6 and D4):** D5 (new numbering) lists each limit of quiet: no live run, unreadable transcript, several live runs, dead pid, and a run busy without progress or in a hung wait.
  D3's Bound names the limits instead of claiming quiet bounds the wait; A2 is restated to match.
- **F9 (D6):** the API error text is cut to one line and 200 characters, as the child's error is, under the fixed prefix.
  The criterion says one line.
- **F10 (D4):** D3 names the own-status seam for the pause flag and adds it to the Constraints, says a failed stat reads as unchanged and a failed decode is the hard error, adds `notice_probe_s: 30` so the probes do not run on every 2 s check, and states that the driver-dead notice is the only signal for a dead driver with an unchanged status file.
- **F11 (D4 and Constraints):** the Constraint says shedengine is unchanged and the row no longer returns a counted Stuck; D3 removes the row's `max_bounces` with that reason.
- **F12 (D2):** removed D2 and the "which D2 keeps current" clause in D1.
  The support log's Interview, Rejected alternatives, Question ledger and Review rounds follow the new numbering, with a note on Review rounds that earlier entries use the old one.
- **F13 (D7):** an awaiting child gets no batten state-changed notice; the driver's `parent_notice` relay is its one channel.
  Goal, D4, D6, the Rejected alternatives and Q12 say so.
- **F14:** removed A1 and A8 from the assumptions; the content stays in D1 and D2.

## Support log

Added Rejected alternatives for the withdrawn turn-end storing, the `PhaseEnteredAt` comparison, the awaiting notice, the seed param, and the unchanged reed enumeration, and Q10–Q12 in the Question ledger.
No Review rounds entry was added for this round; that section is the gate's.

## Judgement calls

- F1, F2, F4 and F10 each needed a design choice the review left open (the cycle's reference time, the marker's home, reed's window addressing, the probe interval and its value).
  The record states each choice; the operator may overrule any of them.
- The Notes for the plan writer gain one line on reed's current window targeting and `remain-on-exit`, which is the cause of F4 and F5.

## Deferred

Nothing deferred.
