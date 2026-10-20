---
round: 1
ledger:
  - key: publish-skip-seam-evaluation-point
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: tree-scope-config-not-open-map
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-category-premise-false
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tree-key-normalisation-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: acceptance-criteria-miss-two-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-omits-loom-round-gate-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: guards-scan-error-undecided
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d4-rationale-overstates-once
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings against the discussion record.
The gating one is the config shape: `tree_scope` must be an open map of `darn.yaml`, and tree scope must turn on and off in one `lyx config darn` call, since a half-set write is refused and rolled back.
The Publish skip seam needs a stated evaluation point at each Publish call, so a PR-Gate rework that adds production code does not inherit an earlier skip.
The rest are wording and coverage corrections: D1's category premise, two missing acceptance criteria, scope's wording on loom's round gate, D4's "once" claim, tree-key normalisation, and the `--guards` scan-error behaviour.
