---
round: 1
ledger:
  - key: d9-task-pass-hub-wide-runproc-registration
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d11-no-map-guard-loses-killed-import
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-machine-sharing-premise-env-layer-and-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-unchanged-map-commit-breaks-audit-filter
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: extra-branch-departure-and-raddle-split
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d7-reachability-bound-no-evidence
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-skip-condition-narrower-than-planupkeep
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d8-compact-write-needs-internal-state
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-lock-placement-and-exit-record-accessor
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-first-prime-call-unowned-with-upkeep-off
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-refusal-d11-import-unordered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d10-layer-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-restates-constraints
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised thirteen findings and no earlier ledger exists.
The gating ones are F1 (BLOCKING, task pass registered hub-wide in `runproc`), F2 (killed import loses the rest), F3 (machine-sharing premise false in two cases) and F4 (audit drops maps left at an older commit).
F5 is a `decision` for the parent: D1 departs from the board Idea's extra branch and junction, and the raddle split follows from that.
F6 to F13 are local corrections to D4, D6, D7, D8, D10 and D11 wording or bounds.
