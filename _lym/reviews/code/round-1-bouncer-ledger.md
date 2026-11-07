---
round: 1
ledger:
  - key: clear-before-command-no-activity-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: confirm-pasted-any-turn-start-fails-command
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: map-delivery-error-taken-as-prompt-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: clear-box-before-command-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: creel-docs-omit-taken-as-prompt-outcome
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-ttl-omits-open-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: template-ttl-comment-omits-open-validation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all open.
Two are gating: the command-paste clear sends its key without a fresh operator-activity check or box read,
and `confirmPasted` fails a command on any later turn start instead of one matching the pasted text.
Two LOW design findings are test gaps for the same change, and three consistency findings are doc and comment updates for the open path and the third inject outcome.
