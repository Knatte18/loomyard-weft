---
round: 1
ledger:
  - key: hubgeom-doc-parent-readers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: parent-resolution-duplicated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: compacted-since-duplicate-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three consistency findings: F1 the hubgeom doc misstates which tellers read the parent, F2 the duplicated resolve-parent block, F3 a duplicate `CompactedSince` test.
None is gating.
