---
round: 1
ledger:
  - key: foreign-box-not-nothing-typed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: start-delivery-not-retried
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: placeholder-acceptance-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: idle-notice-retry-unseen
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: unreadable-box-after-paste-submit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: foreign-box-error-identity
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: bracketed-paste-assumed
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: unknown-readiness-box-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: tmux-invocation-undercount
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: draft-text-in-orch-notice
    rounds: [1]
    status: open
    class: design
    severity: NIT
---

Round 1 raised eleven findings, all open.
The gating cluster is the delivery semantics of the foreign-box error: whether it means nothing was typed (F1), what the start delivery and idle-phase notices do under it (F2, F4), and the unbounded placeholder acceptance (F3).
The rest are gaps in stated outcomes, assumptions and acceptance criteria.
