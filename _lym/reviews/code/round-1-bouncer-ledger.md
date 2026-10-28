---
round: 1
ledger:
  - key: gate-slot-background-stale-parity-pair
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seam-test-cover-wider-than-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (`pattern/PATTERN-gate-slot.md:28` names only `Validate` and `CheckRelevant` for the `ValidateLoomDiscussion` pair) and F2 (`coversSeamTest` matches the test file under every group).
Both are LOW `consistency` with local fixes the finding text already fixes.
