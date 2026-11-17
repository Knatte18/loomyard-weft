---
round: 1
ledger:
  - key: hook-test-spawns-at-tier0
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: argument-cap-off-by-one
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card10-misses-subagent-terms-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: override-exception-trusts-unmarked-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: render-empty-root-branch-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: above-root-claude-md-blocks-launch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: config-dir-failure-outcome-missing
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: target-doc-goes-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-background-listed-as-extends
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hook-run-left-by-killed-prepare
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: hook-env-lacks-claude-project-dir
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, none from an earlier round.
The gating ones are F1 (hook test at tier0, BLOCKING) and F2 (argument cap off by one, MEDIUM design).
F6 and F10 change a refusal and a process-ownership contract, so their fixes need a fresh review.
The rest are local corrections the finding text already fixes.
