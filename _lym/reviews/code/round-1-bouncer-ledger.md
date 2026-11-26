---
round: 1
ledger:
  - key: teardown-spins-when-pushes-off
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: teardown-push-level-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pusher-restart-calls-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pusher-lock-and-conditions-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-prepush-check-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unrelated-slack-relay-test-change
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: pruned-ref-live-branch-push-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-takeover-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: redundant-commit-status-seam-row
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 opened every key above.

- The one behaviour defect is in Worktree-Teardown: with pushes skipped by environment, the level read never turns true and the pair is never removed.
  Its fix adds a pushes-off removal path and bounds the loop with a new Stuck outcome.
- The other MEDIUM keys are coverage gaps around the background pusher: the teardown wiring, the calls that start a pusher, and the real lock and stop conditions.
  Covering the start calls asks for new injection seams.
- The remaining keys are stale comments, a test change outside the task's scope, untested branches and one redundant test row.
