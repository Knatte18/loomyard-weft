---
round: 2
ledger:
  - key: dropped-archive-tag-tests
    rounds: [1]
    status: resolved
  - key: summary-counts-stale-after-fix
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one BLOCKING scope finding: dropped tests that pin archive tags.
The fix commits restored or folded them, and round 2 confirmed no removed test asserts a tag without a kept test pinning it.
Round 2 raised one LOW consistency finding: the change description's counts and kept list predate those fix commits.
