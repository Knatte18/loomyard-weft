---
round: 1
ledger:
  - key: fixhold-way-forward-names-only-failing-subject
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rewriteoverorigin-wraps-nil-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopt-help-describes-a-wait-the-code-does-not-do
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings.
F1 (`FixHold`'s way forward names only the failing subject, so a multi-subject hold gets a re-run the same-set rule refuses) is gating.
F2 (`rewriteOverOrigin` wraps a nil error when the branch has no upstream) and F3 (the adopt help says a plain adopt waits where the code refuses) do not gate.
Each fix text fixes its result, so all three are mechanical.
