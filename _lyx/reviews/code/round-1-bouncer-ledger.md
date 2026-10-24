---
round: 1
ledger:
  - key: retired-spelling-scan-misses-right-of-false-and
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-test-speed-transitional-guard-tag
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pattern-test-isolation-retired-tag-sets
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: loom-plan-spec-older-spellings-types-load
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 (design, MEDIUM) is the scan's `Eval` short-circuit hiding a retired spelling behind a false `&&` term.
F2 and F3 (consistency, MEDIUM) are PATTERN background files still describing retired tag spellings.
F4 (consistency, LOW) is a stale sentence in the loom plan spec.
Each finding text fixes its own correction, so all four are mechanical.
