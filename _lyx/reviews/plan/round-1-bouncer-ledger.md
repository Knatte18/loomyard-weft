---
round: 1
ledger:
  - key: fast-forward-weft-bypasses-anchor-stops
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: owned-elsewhere-way-forward-loops
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: apply-failure-after-commit-no-end-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: anchor-validation-refusals-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: unreachable-invalid-marker-teardown-fallbacks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fabricengine-write-allowlist-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: drop-local-misses-unpushed-reanchor
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: create-at-anchor-engine-path-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: recovery-deletions-bypass-destroy-chokepoint
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: owner-refusal-exemption-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: create-record-removal-untimed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-docs-marker-on-weft-main
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: checkout-standing-record-refusal-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 was the first review; the ledger is new.
One BLOCKING finding: a landing's weft fast-forward bypasses the marker comparison and the move-record stop.
Six more `design` findings at MEDIUM cover the owned-elsewhere way forward, the post-commit Apply failure, unreachable teardown fallbacks, drop-local's comparison, the create-at-anchor engine path and the recovery's deletions.
The rest are consistency, scope and decision findings, two of which need a decision-record entry through the parent.
