---
round: 3
ledger:
  - key: d12-non-exec-calls-outside-cell-model
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-scan-reaches-probe-literals
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d12-read-again-path-resolution-unstated
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d12-codex-breach-rules-undefined
    rounds: [1, 2]
    status: resolved
  - key: d12-question-tool-name-unverified
    rounds: [1, 2]
    status: resolved
  - key: d1-parity-test-misses-non-seatengine-producers
    rounds: [1, 2]
    status: resolved
  - key: assist-trial-schedule-parent-call
    rounds: [1, 2]
    status: resolved
  - key: d12-nudge-block-reason-marks-call-failed
    rounds: [1]
    status: resolved
  - key: d4-hand-written-mark-unheld-inline-prompt
    rounds: [1]
    status: resolved
  - key: d10-no-writes-bound-over-writing-prepare
    rounds: [1]
    status: resolved
  - key: first-criterion-counts-uncompared-d6-d9
    rounds: [1]
    status: resolved
  - key: d8-hook-timeout-and-input-unstated
    rounds: [1]
    status: resolved
  - key: d5-developer-instructions-no-size-bound
    rounds: [1]
    status: resolved
  - key: d3-bound-misses-edited-engine-copy
    rounds: [1]
    status: resolved
  - key: d5-order-lists-d8-items-twice
    rounds: [1]
    status: resolved
  - key: d9-criterion-misses-claude-md-beside-agents-md
    rounds: [1]
    status: resolved
  - key: d9-doc-fact-no-version
    rounds: [1]
    status: resolved
  - key: d2-allowlist-reason-held-by-review-only
    rounds: [1]
    status: resolved
  - key: d1-manifest-record-removed-with-run-dir
    rounds: [2]
    status: resolved
  - key: d4-default-engine-unresolved-from-model-choice
    rounds: [2]
    status: resolved
  - key: d1-shared-heddle-targets-uncompared
    rounds: [2]
    status: resolved
  - key: d12-deny-pretooluse-setting-unplaced
    rounds: [2]
    status: resolved
  - key: d12-codex-chain-key-unnamed
    rounds: [2]
    status: resolved
  - key: d9-override-copy-stale-on-import-change
    rounds: [2]
    status: resolved
  - key: d10-size-check-omits-plugin-session-start
    rounds: [2]
    status: resolved
---

Round 1 raised fifteen findings and round 2 eleven.
Round 3 raised three, down from eleven, and none of them re-raises an earlier key.
Every earlier key is resolved, four of them by the review's explicit evidence: the question deny, the manifest record, D4's engine and the shared heddle targets.

The three open keys are new facets of areas earlier rounds touched.
The BLOCKING one is that D12's call model covers only `exec` custom tool calls, while the rollout also holds top-level `function_call` items such as `spawn_agent` and `send_message`.
The MEDIUM one is that the D1 and D4 producer scan reaches two probe specs in `bouncer.go` that launch nothing.
The LOW one is that D12's read-again rule names no path resolution or modification-time test.
