---
round: 1
ledger:
  - key: vacuous-unknown-field-undecodable-fixtures
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: stale-strict-decode-comment-batten-overview
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: missing-space-unusualfiles-doc-comment
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three non-gating findings (F1 to F3), all test or comment hygiene after the tolerant-decode change.
F1 maps to the vacuous-fixtures key, F2 to the stale-comment key, F3 to the missing-space key.
