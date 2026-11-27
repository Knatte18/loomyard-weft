---
round: 1
ledger:
  - key: off-mode-record-outside-lock
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-window-start-under-next
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: report-update-builder-missing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: probes-and-recordings-deferred-to-implementation
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: placement-child-registration-and-slug-readers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: script-runner-calls-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unknown-worktree-row-misses-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: window-command-skipped-on-child-exits
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tier-table-extends-and-stale-tier-doc
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: mixed-tier-tests-name-no-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: query-within-and-locked-places-read-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reedcli-window-file-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-package-tree-misses-kwinplace
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: recordopen-keeps-settledby
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: undecodable-places-heal-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: vanished-home-branch-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hidden-verb-bound-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: contract-test-covers-part-of-recordings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opens every key.
The weight is in the child flow and planner cards: the lock scope of the mode `off` record, the prime's start desktop,
and the missing builder of the report update.

The deferred KWin probes and operator recordings are a call for the parent.
Its outcome may reopen decisions in the decision record, and may need a fresh Plan-Write,
since the overview outside its Card Index is frozen.
