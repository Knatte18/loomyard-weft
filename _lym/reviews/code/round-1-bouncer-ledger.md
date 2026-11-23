---
round: 1
ledger:
  - key: finalize-parent-fetch-no-retry
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: run-start-reconcile-single-attempt
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: transport-phrases-match-permanent-refusals
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wait-production-path-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wait-order-before-recheck-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: seam-docs-omit-deps-wait
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weft-push-backoff-unverified
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: sideband-test-row-pins-two-phrases
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.

Two BLOCKING scope gaps against the board entry: the Finalize parent upstream fetch (F1) and the run-start and catch-up origin reconciles (F2) still make one attempt with no backoff.
Two MEDIUM design findings: the new transport phrases misclassify permanent 4xx refusals (F3), and the real-clock wait path and wait-error exit have no test (F4).
The LOW and NIT findings cover an unpinned wait order (F5), seam docs missing `Deps.Wait` (F6), unverified weft push backoff (F7) and a two-phrase test row (F8).
