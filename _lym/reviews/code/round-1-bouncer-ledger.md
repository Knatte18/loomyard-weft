---
round: 1
ledger:
  - key: reply-to-slack-mail-skips-relay-check
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: relay-receiver-guard-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: delivered-false-field-absent
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: bridge-read-failure-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: relay-down-warning-overpromises
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: external-sender-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-slack-role-file-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: change-description-misplaces-turn-hook
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, all open.
The gating one is the unpinned relay-receiver guard (design, MEDIUM).
The reply path skipping the relay check is MEDIUM but scope-class.
The rest are LOW: a missing `delivered` field, a raw bridge read error, an overpromising warning, an unpinned EXTERNAL sender, a stale role file and a misplaced turn hook in the change description.
