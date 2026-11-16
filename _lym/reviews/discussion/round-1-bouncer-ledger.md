---
round: 1
ledger:
  - key: d12-nudge-block-reason-marks-call-failed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d12-codex-breach-rules-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d12-question-tool-name-unverified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d1-parity-test-misses-non-seatengine-producers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-hand-written-mark-unheld-inline-prompt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-no-writes-bound-over-writing-prepare
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: first-criterion-counts-uncompared-d6-d9
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: assist-trial-schedule-parent-call
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d8-hook-timeout-and-input-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-developer-instructions-no-size-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d3-bound-misses-edited-engine-copy
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-order-lists-d8-items-twice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-criterion-misses-claude-md-beside-agents-md
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-doc-fact-no-version
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d2-allowlist-reason-held-by-review-only
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
Six design findings at MEDIUM gate the round: the D12 nudge channel, the D12 Codex breach rules, the D12 question-tool name, the parity test's producer coverage, D4's mark holder, and D10's no-writes bound.
The D12 channel, breach-rule, D1, D4 and D10 fixes change a decision or contract and need a fresh review.
The D12 tool-name fix records a fact and the matcher that follows from it.
One decision-class finding, the assist-trial schedule, waits on the parent.
