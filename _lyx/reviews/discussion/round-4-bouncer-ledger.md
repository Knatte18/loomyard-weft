---
round: 4
ledger:
  - key: restart-lookup-adopts-earlier-step-inject
    rounds: [1]
    status: resolved
  - key: fresh-launch-leaves-hold-and-queued-injects
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: color-plugins-steps-escape-bound
    rounds: [1]
    status: resolved
  - key: read-reply-lost-on-restart
    rounds: [1]
    status: resolved
  - key: step-events-offset-at-queueing
    rounds: [1, 2]
    status: resolved
  - key: auto-compaction-reload-unheld
    rounds: [1]
    status: resolved
  - key: detached-notifier-refused-by-identity-rule
    rounds: [1, 2]
    status: resolved
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
    rounds: [2, 3]
    status: resolved
  - key: stop-stamp-lost-between-reads
    rounds: [3]
    status: resolved
  - key: scope-omits-stop-time-shuttle-changes
    rounds: [3]
    status: resolved
  - key: d6-retry-trigger-untyped
    rounds: [3]
    status: resolved
  - key: refused-notice-no-durable-copy
    rounds: [3]
    status: resolved
  - key: support-log-stale-sender-rule
    rounds: [3]
    status: resolved
  - key: withdraw-unlisted-receiver-unstated
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: fresh-launch-inject-ttl-reason-wrong
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resume-noticed-mark-cleared-only-by-start
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised 18 keys, round 2 narrowed to four MEDIUM design findings, and round 3 raised six non-BLOCKING findings.
Round 4 raised four findings, none BLOCKING and no design finding above LOW.
The round-3 keys were not raised again; the review rejected the Stop-stamp and D6 retry-trigger concerns, so they are resolved.
Still open: the unstated handling of an unlisted receiver by the first withdraw (F1, MEDIUM, decision), the failed-release gap in the fresh-launch bound (F2, LOW), the TTL reason for the exposed inject (F3, LOW) and the resume-noticed mark cleared only by `lyx orch start` (F4, LOW).
