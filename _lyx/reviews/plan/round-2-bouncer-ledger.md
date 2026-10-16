---
round: 2
ledger:
  - key: card5-deletes-anchors-under-uncovered-tests
    rounds: [1]
    status: resolved
  - key: render-policy-header-unrewritten
    rounds: [1]
    status: resolved
  - key: reedcli-smoke-headers-keep-below-parent
    rounds: [1]
    status: resolved
  - key: policy-test-file-emptied-not-deleted
    rounds: [1]
    status: resolved
  - key: card5-lists-unchanged-test
    rounds: [1]
    status: resolved
  - key: card2-breaks-update-strand-test
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: stale-file-headers-by-recall
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card5-lists-multiplexer-contract-unchanged
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stacked-adds-subtest-outlives-header
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five findings are not raised again in round 2 and are marked resolved.
Round 2 raised four new keys.
The MEDIUM one is scope-class: card 2 removes the size refusal that `TestUpdateStrandLocked` still pins.
The design-class one is LOW: file headers are rewritten only where a card remembers them.
Two LOW consistency findings cover an Edit target with no stated change and a smoke subtest left behind by a header edit.
