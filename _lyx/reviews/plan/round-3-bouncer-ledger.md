---
round: 3
ledger:
  - key: capture-job-kills-step-process
    rounds: [1]
    status: resolved
  - key: drift-verify-probe-after-mechanical-steps
    rounds: [1]
    status: resolved
  - key: drift-verify-step6-trigger-too-wide
    rounds: [1]
    status: resolved
  - key: card14-misses-routing-test-and-phase-prose
    rounds: [1]
    status: resolved
  - key: dirty-tree-way-forward-at-drift-await
    rounds: [1]
    status: resolved
  - key: drift-baseline-commits-capture-logs-midwrite
    rounds: [1]
    status: resolved
  - key: extends-on-rewritten-refs
    rounds: [1]
    status: resolved
  - key: runs-spent-refusal-unreached
    rounds: [1]
    status: resolved
  - key: accept-agent-attribution-by-running-status
    rounds: [1]
    status: resolved
  - key: startcapture-errors-no-way-forward
    rounds: [1]
    status: resolved
  - key: gatecli-doc-gate-key-example-missed
    rounds: [1]
    status: resolved
  - key: capture-command-two-audiences
    rounds: [1]
    status: resolved
  - key: step7-resume-reads-unkept-stuck-reason
    rounds: [2]
    status: resolved
  - key: resume-test-time-format-mismatch
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: gate-halt-cleared-before-gate-runs
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dirty-read-checkout-unspecified
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: gate-halt-field-has-no-reader
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: equal-head-gate-skip-row-unreached
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: halt-named-at-left-set
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, all resolved by round 2.
Round 2 raised one BLOCKING design finding about the resume discriminator, which the fix moved onto scope-record marks; round 3 did not re-raise it.

Round 3 raised six new findings, all on the scope-record marks and the verify producer's resume steps.
The BLOCKING one is the missing time format on `halt_named_at` and `gate_halt_at`; the proposed fix is a stated format plus a test row.
The MEDIUM design finding reverses round 2's clear-before-gate ordering and needs a fresh review.
The rest are a dirty-read checkout choice, a dead field, an unreached equal-head row and a leftover time.
