---
round: 1
ledger:
  - key: sampler-plan-dir-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: overview-regression-rows-omit-sampler
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 (`sampler-plan-dir-untested`) says the `hasPlan` condition in `internal/loomcli/wiring.go` has no test for the plan directory handed to the Drift expect agent.
F2 (`overview-regression-rows-omit-sampler`) says the regression-rows section of `docs/overview.md` omits the sampler's scope gate and `Sampler-Session` ordering.
Both fixes are local and the review text already fixes their result.
