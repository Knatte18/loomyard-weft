---
round: 1
ledger:
  - key: adoptcli-doc-says-pause-holds-adopt-check
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: no-test-drives-root-prerun-into-gate
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tmuxkit-literal-shares-string-with-pinning-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all with local fixes the review text fixes.
The adopt doc in `internal/adoptcli/doc.go` still says a pause holds an adopt check, which the inverted gate no longer does.
No test drives the real root's pre-run into the gate.
The tmuxkit literal for the run-process mark and the test pinning it share one string.
None is gating, so the loop converges.
