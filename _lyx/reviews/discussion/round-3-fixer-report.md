# Round 3 fixer report

Every finding is fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.
The review was saved before the prior rounds' files were read; the prior rounds' fixes (round 2's F1–F10) stand as written, and no fix here reverses one.

## Changed

- **F1 (D4, Goal, Open risks):** an awaiting child with a `parent_notice` gets batten's notice when its driver strand has ended or 3 minutes passed with no stop report, since no relay can then have happened; a live, reported driver whose relay send failed stays unannounced, stated in D4's Bound and in Open risks.
  The Goal, an acceptance criterion, Q12, Q18 and a Rejected alternative follow.
- **F2 (D4):** `since` is the status file's modification time at the check that first saw the state, kept in the notice's episode key; the report-time filter and the 3 minutes use it.
  The child's history cannot supply it, since a pause or a producer failure appends no entry.
  Rationale, an acceptance criterion, the Interview, Q19 and a Rejected alternative follow.
- **F3 (D3):** the baseline modification time is stat'ed before the call's status read, and any difference triggers the decode; an acceptance criterion and Q20 are added.
- **F4 (D7):** the chained `set-option -w remain-on-exit off` targets the new window by its exact name, the id being for later ops, and a window whose chained option failed is killed by the printed id.
  The behavior was verified on tmux 3.6 on an isolated socket: untargeted, the option landed on the current window; targeted by `=<session>:=<name>`, on the new window only.
  The acceptance criterion now reads the option back on both windows; Q21 and a Rejected alternative follow.
- **F5 (D7):** the strand window resolves from the Selvage pane's window id, then a live strand pane, and only then the current window; the no-batten-window-read claim holds while a Selvage or strand pane is live, and the remaining case (no live pane, as at first `up`) is stated.
  The seam is named as the replacement of `exactSessionWindowTarget`, passed to the engine-less pane listing; the acceptance criterion and the Notes for the plan writer follow.
- **F6 (D4):** `QueueNotice` is not called while the orch state records no strand (one Warn per episode, checked again once per `notice_probe_s`), and a failed queueing is retried at most 3 times; an acceptance criterion and a Rejected alternative follow.
- **F7 (D6, A7, Open risks):** the watched marker is written only by a batten run in the prime, since its notices go to the prime's orch and a driver's parent is the orch of the worktree its pair was created from; a batten from another worktree leaves its driver on today's rule.
  A7, an Open risk, the acceptance criterion, Q22 and a Rejected alternative follow.
- **F8 (D3):** the file reads run on every check; every read that costs a process or a multiplexer round trip (driver liveness wherever read, activity, wait markers, the not-parked resume retry) runs at most once per `notice_probe_s`.
  The done arm's `Done` and the driver-ended trigger follow the driver's end within that interval; the acceptance criterion, a Note for the plan writer, Q23 and a Rejected alternative follow.
- **F9 (D3, Goal, criteria):** the acceptance criterion drops "with its time"; the pause return's `Path` is `pause requested at <time>` so a second pause is its own entry; the failed-stat return is named as the other non-state history entry in D3 and in the Goal.
  Criteria for the second pause and the failed-stat entry are added, with Q24.

## Support log

Added Rejected alternatives for each fix, Q18–Q24 in the Question ledger, a line under Interview turn 6, and restated Q12.
No Review rounds entry was added; that section is the gate's.

## Judgement calls

- F1, F6 and F7 each needed a choice the review left open: the fallback condition for the awaiting notice (driver ended or no report in 3 minutes), the retry cap of 3 for a failed queueing, and "the prime only" for the watched marker; the record states each one, and the operator may overrule any of them.
- F2 uses the status file's modification time rather than the first-seen clock time, because D3 already stats that file and it survives a batten restart within the episode key.

## Deferred

Nothing deferred.
