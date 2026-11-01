---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: halted-fix-task-blocks-new-filing
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: removal-fails-at-cancel-nothing-left
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: on-land-runs-on-non-owner-machine
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: held-slug-list-wiped-by-check-writes
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cancel-lifts-hold-against-declared-pattern
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: front-write-vs-draft-rule
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: machine-failure-halt-names-loom-resume
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cancelled-run-reads-running-here
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: report-run-list-undefined
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-criterion-contradicts-removal-rule
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: board-edit-moves-front-task-routing
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: halted-names-no-run-record-state
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rejected-alternative-inside-d8
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised ten keys on the fix-task lifecycle, D7's state reading and the front-task writes.
Round 2 does not restate any of them; they are recorded as resolved.

Round 2 raises fourteen new keys against the revised text, all first raised in round 2.
- Fix-task lifecycle: slug reuse meets the done-slug refusal, a halted fix task blocks a new filing, removal stops at cancel's nothing-left refusal, and the held-slug list is wiped by the check's own record writes.
- Ownership and declared PATTERN: the land action runs on a machine that no longer owns the subject, and `lym batten cancel` lifts a hold against the declared `PATTERN-adopt-hold`.
- Front-task write and routing: the draft rule defeats the one-write filing, and board edits (project change, rename) move a front task's routing.
- D7 and D9: the run list is undefined, a cancelled run reads `running_here`, the machine-failure halt names `lym loom resume`, and `halted` names no run-record state.
- Wording: D5's criterion contradicts its removal rule, and a rejected alternative sits inside D8.
