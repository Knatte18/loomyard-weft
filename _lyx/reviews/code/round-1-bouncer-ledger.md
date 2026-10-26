---
round: 1
ledger:
  - key: operator-typed-prompt-turn-held-notice-omitted
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: smoke-fixtures-launch-failing-start-twice
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: keytimesreed-doc-comment-displaced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: start-retry-bounds-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stdin-replay-on-retry-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none gating.
F1 is an unrecorded deviation deferred to the parent; F2 and F3 are consistency fixes; F4 and F5 are LOW test-coverage gaps.
