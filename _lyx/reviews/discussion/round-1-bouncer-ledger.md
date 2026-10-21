---
round: 1
ledger:
  - key: stale-junction-sweep-removes-scratch-link
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: check-ignore-probe-shape-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failed-exclude-write-unbounded-downgrade
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: post-rename-refused-link-breaks-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: rewired-branch-reads-two-ways
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: criterion-7-contradicts-recreated-weft
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dangling-link-classification-resolver
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lifetime-claim-false-on-weft-recreation
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: criteria-miss-recording-and-unwire
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rename-has-no-mutation-kind
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: windows-risk-handoff-no-owner
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 raised eleven findings, all open.
The gating ones are the stale-junction sweep that removes the new link (BLOCKING), the check-ignore probe shape and the unbounded exclude-write failure (MEDIUM).
The MEDIUM consistency findings and the LOW ones are local to the record's wording, criteria and decisions.
