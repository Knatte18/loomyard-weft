---
round: 1
ledger:
  - key: doc-go-retry-once-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pushwarprebasefreeat-doc-never-retried
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pushrecorded-doc-retried-once
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: persisting-rejection-code-bolt-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stop-at-once-no-sleep-assertion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five non-gating findings.
Three are stale doc comments that still describe the removed retry-once behavior (F1, F2, F3).
Two are test coverage gaps: the persisting rejection on the code and bolt sides (F4), and the no-sleep stop on named failures (F5).
