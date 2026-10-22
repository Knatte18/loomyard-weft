---
round: 1
ledger:
  - key: d3-dirty-tree-skips-review
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-mixed-failure-log-misclassified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-provably-claim-vs-d5-reach-gap
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-parent-tip-under-push-skip
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d3-stuck-bound-omits-reroute-cap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-exhausted-resume-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-findings-file-removal-unowned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-nonexistent-allow-table-and-path-form
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-d3-narrative-restatement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so every key is new.
F1 (BLOCKING, design) and F2 (MEDIUM, design) gate the loop.
F3 is a MEDIUM consistency finding whose fix is a local wording change.
F4 and F7 are LOW design findings that change what D3 reads and who owns D4's file removal.
The remaining LOW findings are local text corrections.
