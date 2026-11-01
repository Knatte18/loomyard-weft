---
round: 3
ledger:
  - key: rerun-adopt-holds-fix-task
    rounds: [1]
    status: resolved
  - key: held-runs-fill-max-runs
    rounds: [1]
    status: resolved
  - key: cancel-leaves-draft-front-entry
    rounds: [1]
    status: resolved
  - key: landed-state-pruned-entry
    rounds: [1]
    status: resolved
  - key: fix-slug-collision-between-subjects
    rounds: [1]
    status: resolved
  - key: fronttask-write-vs-task-guard
    rounds: [1]
    status: resolved
  - key: front-on-land-refusal-upsert-only
    rounds: [1]
    status: resolved
  - key: on-land-lost-when-dobby-down
    rounds: [1]
    status: resolved
  - key: halted-fix-task-reports-running-here
    rounds: [1]
    status: resolved
  - key: fix-task-perch-rests-on-default
    rounds: [1]
    status: resolved
  - key: fix-slug-reuse-meets-done-slug-refusal
    rounds: [2]
    status: resolved
  - key: halted-fix-task-blocks-new-filing
    rounds: [2]
    status: resolved
  - key: removal-fails-at-cancel-nothing-left
    rounds: [2]
    status: resolved
  - key: on-land-runs-on-non-owner-machine
    rounds: [2]
    status: resolved
  - key: held-slug-list-wiped-by-check-writes
    rounds: [2]
    status: resolved
  - key: cancel-lifts-hold-against-declared-pattern
    rounds: [2]
    status: resolved
  - key: front-write-vs-draft-rule
    rounds: [2]
    status: resolved
  - key: machine-failure-halt-names-loom-resume
    rounds: [2]
    status: resolved
  - key: cancelled-run-reads-running-here
    rounds: [2]
    status: resolved
  - key: report-run-list-undefined
    rounds: [2]
    status: resolved
  - key: d5-criterion-contradicts-removal-rule
    rounds: [2]
    status: resolved
  - key: board-edit-moves-front-task-routing
    rounds: [2]
    status: resolved
  - key: halted-names-no-run-record-state
    rounds: [2]
    status: resolved
  - key: rejected-alternative-inside-d8
    rounds: [2]
    status: resolved
  - key: fix-slug-number-lost-after-cleanup
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: red-check-over-live-own-fix-task
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fix-try-count-has-no-store
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: awaiting-reads-running-here
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-criterion-disagrees-with-d4-lifters
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised twenty-four keys on the fix-task lifecycle, ownership, the front-task write and routing, and D7 and D9 wording.
Round 3 does not restate any of them; they are recorded as resolved.

Round 3 raises five keys, all first raised in round 3, none above MEDIUM.
- Fix-task lifecycle: the slug number is lost when cleanup removes the run record, a red check over a live own fix task has no outcome, and the fix-try count has no named store.
- Wording: `awaiting` reads as running here, and D3's criterion disagrees with D4's hold lifters.
