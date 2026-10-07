# Round 2 fixer report

Every finding is fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.
The prior round's files were read only after the review was saved; none of its fixes regressed, and F1 and F2 below refine its F13 and F7 resolutions.

## Changed

- **F1 (D4, D6, BLOCKING):** batten's state-changed notice now covers an `awaiting` child whose status carries no `parent_notice` (every PR-Gate hand-off, and a Bouncer escalation whose notice failed to render); with a `parent_notice`, the driver's relay stays the one channel.
  D6 drops the generic escalation line for a watched driver and says so; the Goal, an acceptance criterion, Q12 and a Rejected alternative follow.
- **F2 (D3, D4):** a pending notice is derived from the child's status and the episode marker, so it outlives the call that returned for the state change and is sent by the wait of the next call, its 3 minutes counted from `since`; a pause or error return leaves it pending, and `Done` makes one last attempt, a notice not queued then being lost (Open risks).
  The "before the call returns for any reason" sentence is gone.
- **F3 (D6, D4):** batten writes the watched marker only while the orch state records a strand; a not-queued or failed notice is retried at most once per `notice_probe_s`; D6's Bound says the rule is fixed at the driver's fresh launch (a resume renders nothing), with the adopted and the orphaned cases named.
  Open risks and the acceptance criteria follow.
- **F4 (D7):** reed learns its strand window from the window holding its recorded Selvage pane, with today's current-window target as the fallback, and every window-targeted op goes through that one seam (the window-size, resize-hook, generation and attach-geometry reads included); `new-window` joins the required verbs.
  The acceptance criterion and the Notes for the plan writer follow.
- **F5 (D7, Open risks):** the window is created and its `remain-on-exit off` set in one invocation; the envelope reports a started window, not a started batten, and a lock refusal is read from batten's log; the Open risk now covers the end of the reed session.
- **F6 (D3):** a failed stat ends the wait with a budget-exempt `Stuck`, and the next status read decides (absent as today, unreadable as the hard error).
  A criterion is added.
- **F7 (D3, Goal, criteria):** the change text rides in the pointer's `Path`, because a history entry stores the path, not the reason, and equal outputs fold; a requested pause is named as the one other history entry.
- **F8 (D3):** the cheap file reads and the driver liveness read while a done child or a pending notice waits run on every check; only a running child's liveness, activity and wait-marker probes follow `notice_probe_s`.
- **F9 (D4):** a done notice carries no report path and goes out on the first check that sees the child done, since Worktree-Teardown removes the worktree right after `Done`; the newest-file lookup is gone.
  D6 keeps the driver's message for a `done` stop whose friction failed, which batten's done notice cannot carry; Q14 and a Rejected alternative record it.
- **F10 (D2):** removed the "older than the cycle start" clause and, with it, the `CycleStartedAt` field: the cycle's entry clears every pending request, so the lower bound guarded nothing.
  D2 keeps the mode test and the boundary bound; scope, the Interview, Q3 and the Rejected alternatives follow.

## Support log

Added Rejected alternatives for the all-awaiting exclusion, the done report path, the cycle start field, the stat-as-unchanged rule and the separate `remain-on-exit` call, and Q13–Q17 in the Question ledger; Q3 and Q12 and Interview turn 3 are restated.
No Review rounds entry was added; that section is the gate's.

## Judgement calls

- F1, F3, F4 and F9 each needed a choice the review left open (batten covers awaiting without a notice, the marker's destination condition, the Selvage pane as the window id source, the done notice without a report); the record states each one, and the operator may overrule any of them.
- F10 removes a field the prior round added; the review's fix said to reword the clause, and the field had no other use.

## Deferred

Nothing deferred.
