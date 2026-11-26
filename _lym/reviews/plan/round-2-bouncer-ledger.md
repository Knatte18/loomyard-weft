---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's keys all concerned the run watch engine card and plan-text corrections around it.
Round 2 closes every one of them, one through an added decision-record entry and one on a rebuttal.
One new defect is open, in the run watch engine card:
the task's `run_watch` level is read again after the task worktree is gone, and the inherited value replaces the task's own.
