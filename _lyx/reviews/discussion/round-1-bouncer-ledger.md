---
round: 1
ledger:
  - key: confirmation-states-unknown-passes
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: no-live-pause-when-unowned
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: sleep-deny-wrong-verbs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sleep-deny-bound-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: watch-verb-unguarded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: daemon-liveness-test-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pause-grace-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: deliberation-in-decision-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review of the hibernate-trigger discussion pair.
Two BLOCKING design findings concern the pre-hibernate safety conditions: the confirmation reading uses state names tender's status never reports and lets `unknown` pass, and no live hub pause is required when the daemon owns no entry.
Three MEDIUM design findings concern the sleep-command deny (wrong verbs, unstated match shape and bypasses) and the unguarded hidden `watch` verb.
Four LOW findings are local corrections: liveness test, pause grace, missing acceptance criteria, and deliberation narrative to relocate.
