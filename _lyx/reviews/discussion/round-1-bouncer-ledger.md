---
round: 1
ledger:
  - key: scope-full-accept-resume-loop
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: d4-runproc-false-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: scope-partial-accept-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-drift-baseline-replans-loom
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-loses-baseline
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: drift-await-clock-mismatch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: compare-side-verbs-no-row-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-webster-regression-second-halt
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: drift-verify-placement-parent-align
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: drift-await-done-vs-skipped
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: capture-holds-back-plan-gate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-code-change-uncompared-too-broad
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: e2e-proof-location-sandbox-hub
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ac1-darn-drift-await
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: lyx-exclude-existing-behaviour
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so no key recurs.
Keys map to F1 through F15 in order.
The two BLOCKING keys are the scope-full accept-and-resume contradiction across D2, D6 and D7, and D4's false premise about process-tree ending.
Five MEDIUM `design` keys follow, covering the unbounded partial acceptance, the loom re-plan on goto, the adopt losing the baseline, the clock mismatch and the unguarded compare verbs.
The two `decision` keys go to the run's parent.
The remaining LOW and NIT keys are local text corrections.
