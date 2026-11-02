---
round: 1
ledger:
  - key: deploy-gate-held-root-and-owned-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: read-deploy-state-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-cli-cobra-adoptcli-line-omits-imports
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-adopt-bullet-omits-deploy-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: run-adopt-deploy-result-mapping-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-deploy-row-omits-mutation-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 (first ledger): F1 to F6 map to the keys above in order.
Both MEDIUM findings are missing test coverage of the deploy step in `internal/adoptcli`; the other four are a rule-document line, an overview sentence and two further test gaps.
All fixes are local corrections whose result the finding text fixes.
