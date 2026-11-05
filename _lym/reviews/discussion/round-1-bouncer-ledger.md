---
round: 1
ledger:
  - key: d11-d10-dobby-land-rewrite-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hand-member-run-into-ready-beam
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-repair-cap-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merged-pr-older-head-passes-ready
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-pr-refusal-classes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pawl-finalize-scratch-pr-orphaned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: push-lock-point-parent-decision
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d13-tracked-changes-no-commit
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d6-close-comment-false-on-merge-parent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-misstates-ready-record-ender
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d13-reopen-request-left-behind
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-describe-overwrites-summary
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d11-arbiter-eligibility-empty-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d13-judged-first-term-misuse
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retry-windows-unnamed-unvalued
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-rationale-hides-requirement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: restated-assumptions-and-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first ledger; nothing is carried forward.
The BLOCKING key `d11-d10-dobby-land-rewrite-premise` (review F1) is the crux: crash recovery and the goto bound rely on a dobby that its persisted land episode keeps from asking again.
Five more design keys (F2 to F6) sit around the ready window and the in-flight move.
`push-lock-point-parent-decision` (F7) goes to the run's parent.
The remaining keys are local corrections and relocations.
