---
round: 1
ledger:
  - key: beginbatch-last-batch-flag-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: detached-runner-main-packages-words-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: stencil-status-rules-miss-last-batch-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: withmainpackages-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: modulemainpackages-e-behavior-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), none of class `design`.
F1 and F2 are MEDIUM test gaps for the `--main-packages` wiring, F3 is a stencil consistency gap, and F4 and F5 are LOW test gaps.
All five are local corrections the finding text already specifies.
