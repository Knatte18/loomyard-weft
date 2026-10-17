---
round: 2
ledger:
  - key: restart-lookup-adopts-earlier-step-inject
    rounds: [1]
    status: resolved
  - key: fresh-launch-leaves-hold-and-queued-injects
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: color-plugins-steps-escape-bound
    rounds: [1]
    status: resolved
  - key: read-reply-lost-on-restart
    rounds: [1]
    status: resolved
  - key: step-events-offset-at-queueing
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: auto-compaction-reload-unheld
    rounds: [1]
    status: resolved
  - key: detached-notifier-refused-by-identity-rule
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hub-wide-strand-fault-blocks-cycles
    rounds: [1]
    status: resolved
  - key: role-read-all-undoes-held-read-from
    rounds: [1]
    status: resolved
  - key: strand-signed-notice-return-bound
    rounds: [1]
    status: resolved
  - key: reply-form-refused-by-cli
    rounds: [1]
    status: resolved
  - key: pause-resume-line-can-expire
    rounds: [1]
    status: resolved
  - key: revive-inject-sender-unstated
    rounds: [1]
    status: resolved
  - key: concurrent-revives-both-inject
    rounds: [1]
    status: resolved
  - key: revive-report-claims-stale
    rounds: [1]
    status: resolved
  - key: guid-to-name-unstated
    rounds: [1]
    status: resolved
  - key: orch-status-no-hold-send-fields
    rounds: [1]
    status: resolved
  - key: d2-stencil-reason-misreads-open-point
    rounds: [1]
    status: resolved
  - key: relaunch-old-note-request-reminded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised 18 keys.
Round 2 raised four MEDIUM design findings; the rest of round 1's keys were rejected by the fresh review or not raised again, so they are resolved.
Three round-2 findings reopen round-1 keys: the failed withdraw at relaunch (F2), the D6 in-pane test (F3) and the sending-time filter (F4).
The fourth, the unread note-request mail surviving a relaunch (F1), is new.
