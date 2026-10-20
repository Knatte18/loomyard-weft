---
round: 1
ledger:
  - key: attach-gate-path-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tackler-ungated-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: attach-doc-comment-not-standalone
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three non-gating findings, all local corrections whose text fixes the result.
F1 maps to `attach-gate-path-untested`, F2 to `tackler-ungated-unpinned` and F3 to `attach-doc-comment-not-standalone`.
