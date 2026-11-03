---
round: 1
ledger:
  - key: ended-phases-literal-copy
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: hibernate-daemon-live-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hibernate-dir-missing-at-watch-open
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resuming-takeover-skips-reconcile-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancelled-sequence-keeps-carrier-busy
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: stale-daemon-comment-cleanup-test
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none seen before.
The blocker is the repeated terminal phase literals in `fabriccli`, F1.
The untested `hibernateDaemonLive` ended-phase rule, F2, is the gating `design` finding.
The other four are delay or wording issues of LOW or NIT severity.
