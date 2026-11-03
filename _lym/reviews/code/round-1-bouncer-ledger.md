---
round: 1
ledger:
  - key: origin-task-branch-delete-failure-unreported
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-handover-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: notice-remote-teardown-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: teardown-child-pause-exemption
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: teardown-child-registered-as-push-role
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: default-way-forward-wrong-for-kind
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-job-no-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: remove-step-archive-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-fabric-git-background-old-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first ledger; nothing carries forward.
Four MEDIUM `design` keys gate: the unreported origin task-branch delete failure, the untested detached handover, the untested notice function, and the teardown child's missing pause exemption.
The five LOW keys cover the child's process role, the default way forward by job kind, an unreadable job, and two stale descriptions of the archive-tag order.
