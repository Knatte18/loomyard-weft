---
round: 1
ledger:
  - key: card7-attach-preflight-docs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card4-gitrepo-doc-leaf-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-bolt-empty-subtest-redundant
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card8-loop-scan-trips-on-doc-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-spawn-check-preempted-by-runner
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings: stale docs in cards 7 and 4, a redundant Bolt subtest in card 3, a self-tripping scan in card 8, and an unstated backstop role in card 6.
None is BLOCKING and none is a gating `design` finding at MEDIUM or worse.
Each fix is a local correction the finding text already fixes.
