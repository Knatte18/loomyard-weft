---
round: 2
ledger:
  - key: origin-task-branch-delete-failure-unreported
    rounds: [1]
    status: resolved
  - key: detached-handover-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: notice-remote-teardown-untested
    rounds: [1]
    status: resolved
  - key: teardown-child-pause-exemption
    rounds: [1]
    status: resolved
  - key: teardown-child-registered-as-push-role
    rounds: [1]
    status: resolved
  - key: default-way-forward-wrong-for-kind
    rounds: [1]
    status: resolved
  - key: unreadable-job-no-notice
    rounds: [1]
    status: resolved
  - key: remove-step-archive-comment-stale
    rounds: [1]
    status: resolved
  - key: pattern-fabric-git-background-old-order
    rounds: [1]
    status: resolved
  - key: teardown-signal-or-job-write-no-notice
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-job-notice-omits-discard
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: non-archive-failure-reported-as-archive
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised nine keys, four of them gating MEDIUM `design`.
Round 2's review found eight of them fixed: the origin task-branch delete report, the notice tests, the pause exemption, the process role, the default way forward by kind, the unreadable-job notice, and the two stale archive-order descriptions.

`detached-handover-untested` stays open.
The `childHold.finish` rows landed, but `spawnTeardownHold` and the teardown child verb's `RunE` are still unreached by any test, so F1 reopens the key at MEDIUM `design`.

Three LOW keys are new in round 2: a signal-ended child or an unwritable job leaves no creel notice, the unreadable-job notice omits the discard way forward, and a non-archive failure is reported as an archive failure.
