---
round: 1
ledger:
  - key: no-hold-reread-between-check-and-beam-steps
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: fix-task-run-not-moved-under-take-hold
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: parent-warning-left-unreachable
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card-9-batch-gate-meets-importerless-adoptrecord
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixhold-clearhold-success-over-refused-replay
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: takeover-retry-drops-parent-records-commits
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: one-subject-ways-forward-under-multi-subject-holds
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobby-runs-pattern-summary-line-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: holdwrite-file-header-falsified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: endsubject-doc-names-adopt-worker
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card-8-misstates-fronttask-check-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: replay-parent-hold-lost-branch-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parsereplayholds-duplicates-parseowners
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
Three BLOCKING findings stand: a missing hold re-read after the check, a fix task never moved under a take-hold, and `parentWarning` left unreachable.
Four MEDIUM `design` findings gate as well: card 9's batch gate against card 10, `FixHold` and `ClearHold` over a refused replay,
the takeover retry on the parent's records branch, and one-subject ways forward under multi-subject holds.
Six LOW findings are local corrections.
