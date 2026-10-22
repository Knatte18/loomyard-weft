---
round: 1
ledger:
  - key: card10-static-prose-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: card7-warn-repair-unreachable-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card11-resume-noticed-clear-never-retried
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reed-status-omits-elsewhere
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card11-no-id-branch-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: strandremovable-doc-comment-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card9-restates-awakesince
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-impossible-state-row
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: defensive-branches-for-impossible-wiring
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings: F1 to F9, in the order of the keys above.
F1 is the only BLOCKING one and F2, F3 are the gating MEDIUM design findings.
F2 and F3 change what the plan promises (a Warn way forward, a retry), so their fixes need a fresh review.
The rest are local corrections the finding text already fixes.
