---
round: 1
ledger:
  - key: gateslot-watchdog-line-bundled-in-gatecli-card
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: rerun-skip-no-stated-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card2-misses-comments-made-wrong
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card4-replace-all-contradicts-rename-target
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: timed-out-step-not-reported-as-not-rerun
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: enclosing-variable-bound-overstates-ignored
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: sampler-failure-branch-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: negative-key-refusal-wording-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eight findings, none recurring.
The gating one is F1, the gateslot watchdog-line symbols living inside the gatecli watchdog card.
F2 is the next most severe (MEDIUM, `scope`, not gating).
