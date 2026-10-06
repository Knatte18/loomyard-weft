---
round: 3
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
    status: resolved
  - key: card18-leftover-grouping
    rounds: [2]
    status: resolved
  - key: stale-card-number-in-procedure
    rounds: [3]
    status: open
    class: consistency
    severity: BLOCKING
  - key: skill-loading-reason-outside-closed-set
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: loop-exit-ignores-reason-class
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: card16-too-many-concerns
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: card19-leaf-label-mismatch
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 and 2 raised nine findings; round 3 raises none of them again, so all nine are resolved.
The two round 2 design findings were addressed: the procedure now states the keep-reason classes, and the leftover packages were regrouped.

Round 3 raises five new findings.
`stale-card-number-in-procedure` (F1, BLOCKING): step 1 names card 19 as the report-rewriting card, but card 20 does the splice.
`skill-loading-reason-outside-closed-set` (F2): the skill-loading keep reason fits none of the four stated classes, so card 20's check would reject a keep the plan requires.
`loop-exit-ignores-reason-class` (F3, design): step 8 exits on the presence of a directive, not a valid reason.
`card16-too-many-concerns` (F4, design): card 16 bundles seventeen packages; the review proposes a hub-and-geometry card and a git-and-filesystem card, with renumbering.
`card19-leaf-label-mismatch` (F5, LOW): card 19 mislabels `batcher` and `shell` and holds the leaf closed-set rule while two leaf packages sit in card 16.
