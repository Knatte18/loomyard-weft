---
round: 2
ledger:
  - key: d3-dirty-tree-skips-review
    rounds: [1]
    status: resolved
  - key: d1-mixed-failure-log-misclassified
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-provably-claim-vs-d5-reach-gap
    rounds: [1]
    status: resolved
  - key: d3-parent-tip-under-push-skip
    rounds: [1]
    status: resolved
  - key: d3-stuck-bound-omits-reroute-cap
    rounds: [1]
    status: resolved
  - key: gate-exhausted-resume-text-stale
    rounds: [1]
    status: resolved
  - key: d4-findings-file-removal-unowned
    rounds: [1]
    status: resolved
  - key: d6-nonexistent-allow-table-and-path-form
    rounds: [1]
    status: resolved
  - key: scope-d3-narrative-restatement
    rounds: [1]
    status: resolved
  - key: open-risks-name-refused-goto
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-status-read-constraint-wrong-reader
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine keys, led by the BLOCKING `d3-dirty-tree-skips-review`.
Round 2 does not re-raise eight of them, and its focus verdicts find D3, D4 and the parent tip sound, so those are resolved.
The mixed-log key persists in a narrower form, F2: the opaque-failure rule cannot fire beside a parsed identity.
Round 2 adds two new keys: F1, Open risks naming a refused goto (MEDIUM, design), and F3, the PATTERN-fabric-git constraint naming the wrong reader (LOW, consistency).
