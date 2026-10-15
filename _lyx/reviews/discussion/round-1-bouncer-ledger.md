---
round: 1
ledger:
  - key: refshape-planparser-export-missing-from-scope
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: d1-deciding-file-for-moved-symbol
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d2-uses-bullet-contradicts-d1
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: acceptance-criteria-decisions-not-one-to-one
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-silent-on-third-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-rationale-history-overclaim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order above), none gating.
F1 and F2 change what the record decides, so their fixes need a fresh look.
F3 to F6 are local wording corrections.
