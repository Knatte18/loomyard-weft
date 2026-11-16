---
round: 1
ledger:
  - key: pattern-test-speed-background-prime-only
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: nonprime-test-anchor-equals-root
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testmap-long-drops-audit-round
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all with local fixes the review text already specifies.
F1 is the stale PATTERN-test-speed background (lines 221 and 290), F2 the non-prime test that cannot tell the anchor from the worktree root, F3 the dropped audit sentence in the testmap Long.
None is gating.
