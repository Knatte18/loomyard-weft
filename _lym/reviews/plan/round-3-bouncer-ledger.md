---
round: 3
ledger:
  - key: done-run-clear-drops-unread-level-run
    rounds: [1]
    status: resolved
  - key: failed-readchild-seen-times-unspecified
    rounds: [1]
    status: resolved
  - key: stale-land-notice-copy-silences-landed-lines
    rounds: [1]
    status: resolved
  - key: level-unread-skip-also-skips-land-notice-copy
    rounds: [1]
    status: resolved
  - key: checklocalvalues-test-duplicates-runsettings-test
    rounds: [1]
    status: resolved
  - key: gate-hold-process-cap-unlicensed
    rounds: [1]
    status: resolved
  - key: card3-docgo-rewrite-under-extends
    rounds: [1]
    status: resolved
  - key: board-upsert-help-omits-run-watch
    rounds: [1]
    status: resolved
  - key: overview-doc-omits-new-dobby-mail
    rounds: [1]
    status: resolved
  - key: nested-gate-marker-bound-unstated
    rounds: [1]
    status: resolved
  - key: failed-state-write-loses-landed-line
    rounds: [1]
    status: resolved
  - key: failing-loadconfig-row-false-reason
    rounds: [1]
    status: resolved
  - key: loadconfig-doc-omits-range-refusals
    rounds: [1]
    status: resolved
  - key: level-reread-after-worktree-teardown
    rounds: [2]
    status: resolved
---

Round 1's keys concerned the run watch engine card and plan-text corrections around it, and round 2 closed them all.
Round 2 opened one BLOCKING defect in that card:
the task's `run_watch` level was read again after the task worktree was gone.
Round 3 closes it and raises nothing, so no key is open.

Round 3 listed items below its incremental bar without raising them:
the missing `Episode` inputs for a kept child part in `WatchReading`,
a narrow window in the level read past Land,
the autonomy stamp on a landed line posted after teardown,
an untested `watchRecordFailed` hold,
and which gate slot supplies a PID group's line.
