---
round: 2
ledger:
  - key: pre-drive-refusals-no-way-forward-clause
    rounds: [1]
    status: resolved
  - key: busy-batten-lock-hook-keeps-lock-across-retry
    rounds: [1]
    status: resolved
  - key: offline-cancel-reads-no-owner
    rounds: [1]
    status: resolved
  - key: remove-flag-no-stated-bound
    rounds: [1]
    status: resolved
  - key: reached-parent-refusal-marker-loop
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: busy-write-lock-refusal-no-holder-path
    rounds: [1]
    status: resolved
  - key: boardengine-doc-keeps-old-runlock-way-forward
    rounds: [1]
    status: resolved
  - key: spec-stepbusykind-doc-names-one-sentinel
    rounds: [1]
    status: resolved
  - key: battenprestep-kind-mapping-doc-stale
    rounds: [1]
    status: resolved
  - key: batten-lock-seam-busy-path-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: kindbusy-doc-stale-on-no-card
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: busy-hook-retry-skips-driver-lock-probe
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-bundles-read-and-destructive-surfaces
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, one BLOCKING.
Round 2 shows eight of them fixed and gone from the review; the reached-parent marker loop (F1) recurs because the fixer deferred it pending a recorded decision.
Round 2 adds four new keys: the untested busy path of the batten-lock seam (F2), the stale `KindBusy` doc (F3), the busy-hook retry skipping the driver-lock probe (F4) and card 4's bundled surfaces (F5).
The round-1 key about the hook keeping the lock across a retry is resolved, since the plan now states the release; F2 is a separate test gap on the same seam.
