---
round: 2
ledger:
  - key: d12-codex-breach-rules-undefined
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d12-question-tool-name-unverified
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d1-parity-test-misses-non-seatengine-producers
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: assist-trial-schedule-parent-call
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-default-engine-unresolved-from-model-choice
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-shared-heddle-targets-uncompared
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d12-deny-pretooluse-setting-unplaced
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d12-codex-chain-key-unnamed
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d9-override-copy-stale-on-import-change
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d10-size-check-omits-plugin-session-start
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised fifteen findings, six of them gating design findings at MEDIUM.
Round 2 shows most of them settled: the D12 channel, the D4 mark holder, the D10 bound and the rest are resolved.
Three round-1 keys remain open: the D12 breach rules, now BLOCKING because the Codex rollout runs JavaScript exec cells, the D12 question-tool name, and the D1 producer scan.
`assist-trial-schedule-parent-call` is a consistency finding now, since the Added after Discussion entry answers it; its fix drops the Open risks line and resolves Q14.
Round 2 adds seven keys: the manifest record's lifetime, D4's default engine, shared heddle targets in D1, the deny's PreToolUse placement, the Codex chain key, the stale override copy, and D10's size check.
