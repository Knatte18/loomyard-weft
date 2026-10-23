---
round: 1
ledger:
  - key: pre-drive-refusals-no-way-forward-clause
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: busy-batten-lock-hook-keeps-lock-across-retry
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: offline-cancel-reads-no-owner
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: remove-flag-no-stated-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reached-parent-refusal-marker-loop
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: busy-write-lock-refusal-no-holder-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: boardengine-doc-keeps-old-runlock-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spec-stepbusykind-doc-names-one-sentinel
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: battenprestep-kind-mapping-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings: F1 to F9 map in order to the keys above.
F1 is BLOCKING and F2, F6 are gating `design` findings, so the loop continues.
F2 and F5 change a seam contract and a decision, so their fixes need a fresh review; the rest are local corrections the finding text fixes.
