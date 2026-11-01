---
round: 1
ledger:
  - key: rerun-adopt-holds-fix-task
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: held-runs-fill-max-runs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cancel-leaves-draft-front-entry
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landed-state-pruned-entry
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fix-slug-collision-between-subjects
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fronttask-write-vs-task-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: front-on-land-refusal-upsert-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: on-land-lost-when-dobby-down
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: halted-fix-task-reports-running-here
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fix-task-perch-rests-on-default
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so there is no earlier ledger and every key is new and open.

The design fails in three places.
- Fix-task lifecycle: a re-run of `lym adopt` holds the fix task, held runs fill `max_runs`, and cancel leaves a draft front entry.
- D7's state reading: `landed` depends on a done entry that Land prunes, and a halted fix task reads as `running_here`.
- Front-task writes: the slug is not unique per subject, the filing write's relation to the task guard is unstated, the refusal covers `upsert` only, and `on_land` is lost across a dobby restart.

The fix task's perch (LOW) rests on the hub default.
