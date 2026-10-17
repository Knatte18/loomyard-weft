---
round: 1
ledger:
  - key: restart-lookup-adopts-earlier-step-inject
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fresh-launch-leaves-hold-and-queued-injects
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: color-plugins-steps-escape-bound
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: read-reply-lost-on-restart
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: step-events-offset-at-queueing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: auto-compaction-reload-unheld
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-notifier-refused-by-identity-rule
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hub-wide-strand-fault-blocks-cycles
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: role-read-all-undoes-held-read-from
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: strand-signed-notice-return-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reply-form-refused-by-cli
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pause-resume-line-can-expire
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: revive-inject-sender-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: concurrent-revives-both-inject
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: revive-report-claims-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: guid-to-name-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-status-no-hold-send-fields
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-stencil-reason-misreads-open-point
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The gating set is the three BLOCKING keys (restart lookup, fresh-launch leftovers, color and plugins steps unbounded) and the five MEDIUM `design` keys.
The two MEDIUM `consistency` keys (read command, strand-signed return) do not gate.
Keys mapped to findings: F1 to F18 in review order.
