---
round: 3
ledger:
  - key: restart-lookup-adopts-earlier-step-inject
    rounds: [1]
    status: resolved
  - key: fresh-launch-leaves-hold-and-queued-injects
    rounds: [1, 2, 3]
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stop-stamp-lost-between-reads
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: scope-omits-stop-time-shuttle-changes
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: d6-retry-trigger-untyped
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: refused-notice-no-durable-copy
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: support-log-stale-sender-rule
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 18 keys; round 2 narrowed to four MEDIUM design findings, three of them reopenings.
Round 3 raised six findings, none BLOCKING.
F1 reopens the failed-withdraw failure path at relaunch (`fresh-launch-leaves-hold-and-queued-injects`) and the unread old note request (`relaunch-old-note-request-reminded`); both stay open, which makes the verdict CIRCLING.
`step-events-offset-at-queueing` and `detached-notifier-refused-by-identity-rule` were not raised again and are resolved.
The other five round-3 keys are new: the Stop stamp split across reads (F2), the Scope omission for the Stop time (F3), the untyped creel refusal behind D6's retry (F4), the missing durable copy of a refused notice (F5) and the stale sender rule in the support log (F6).
