---
round: 1
ledger:
  - key: capture-job-kills-step-process
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: drift-verify-probe-after-mechanical-steps
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: drift-verify-step6-trigger-too-wide
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card14-misses-routing-test-and-phase-prose
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: dirty-tree-way-forward-at-drift-await
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: drift-baseline-commits-capture-logs-midwrite
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: extends-on-rewritten-refs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runs-spent-refusal-unreached
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: accept-agent-attribution-by-running-status
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: startcapture-errors-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gatecli-doc-gate-key-example-missed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: capture-command-two-audiences
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Keys map to the review's findings in order: F1 to F12.
The two BLOCKING design findings concern the capture job's kill-on-close lifetime and the order of Drift-Verify's live-chair probe.
