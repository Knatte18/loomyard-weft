---
round: 1
ledger:
  - key: resolve-run-id-pr-findings-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: send-back-double-friction-note
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bouncer-amendment-full-input-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: drift-command-mode-loosened-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loosened-trailer-lines-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shedstatus-and-sendbackedge-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prreview-wait-contained-test-duplicate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: recipe-reroute-tests-need-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-drift-and-ask-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: fixture-comment-stuck-always-halt
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: fragment-header-enumerates-consumers
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings and no BLOCKING one.
The only gating finding is the untested `pr-findings` exemption in `resolveRunID`.
The rest are test-coverage gaps, two test-economy marks, a stale overview passage and two comment NITs.
Each fix text names its own result, so all are mechanical.
