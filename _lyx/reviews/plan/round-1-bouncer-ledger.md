---
round: 1
ledger:
  - key: card5-deletes-anchors-under-uncovered-tests
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: render-policy-header-unrewritten
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reedcli-smoke-headers-keep-below-parent
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: policy-test-file-emptied-not-deleted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-lists-unchanged-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none seen before.
The BLOCKING one is that card 5 deletes `render.AnchorTop` and `render.AnchorBelowParent` while four test members still reference them.
Two MEDIUM scope gaps leave stale `below-parent` wording in file headers that the verify grep either catches or exempts.
Two LOW consistency findings cover an emptied test file that should be deleted whole, and a card 5 Edit target whose Intent changes nothing in it.
