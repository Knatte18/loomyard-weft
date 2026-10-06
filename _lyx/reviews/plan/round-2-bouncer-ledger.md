---
round: 2
ledger:
  - key: card3-prunes-testtiming
    rounds: [1]
    status: resolved
  - key: skill-loading-carveout-uncarried
    rounds: [1]
    status: resolved
  - key: card18-testtiming-in-scope
    rounds: [1]
    status: resolved
  - key: card17-catchall-packages
    rounds: [1]
    status: resolved
  - key: card5-undefined-scenario-merge-ref
    rounds: [1]
    status: resolved
  - key: decision13-bounds-uncarried
    rounds: [1]
    status: resolved
  - key: card18-link-integrity-missing
    rounds: [1]
    status: resolved
  - key: keep-reasons-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card18-leftover-grouping
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised seven findings; round 2 raises none of them again, and its settled points confirm the testtiming exclusion, the skill-loading carve-out and the card numbering, so all seven are resolved.
Round 2 raises two new MEDIUM design findings.
`keep-reasons-unbounded` (F1): the prune procedure omits Decision 10's four keep-reason classes and the never-annotate rule.
`card18-leftover-grouping` (F2): card 18 mixes CLI and IDE, friction and pure-leaf packages; the review proposes splitting it in two and renumbering the rerun card.
