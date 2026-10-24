---
round: 3
ledger:
  - key: retagged-tier0-tests-drop-from-impact-loads
    rounds: [1]
    status: resolved
  - key: diff-base-ignores-args-fingerprint-module
    rounds: [1]
    status: resolved
  - key: d8-relaunch-bypass-rules-mismatch-code
    rounds: [1]
    status: resolved
  - key: entry-gate-vs-pause-before-order
    rounds: [1]
    status: resolved
  - key: darn-tree-scope-verify-unaddressed
    rounds: [1]
    status: resolved
  - key: carry-bound-omits-forged-record-and-env
    rounds: [1]
    status: resolved
  - key: zero-tests-failure-trips-on-run-filter
    rounds: [1]
    status: resolved
  - key: discovered-count-check-has-no-owner
    rounds: [1]
    status: resolved
  - key: partial-carry-writes-no-whole-entry
    rounds: [1]
    status: resolved
  - key: d3-darn-lint-base-called-unchanged
    rounds: [1]
    status: resolved
  - key: bare-operand-drops-nonbool-refusal
    rounds: [1]
    status: resolved
  - key: carry-criterion-contradicts-d4-on-run
    rounds: [1]
    status: resolved
  - key: window-force-on-inner-open-terminal-flag
    rounds: [1]
    status: resolved
  - key: d5-rationale-deliberation-narrative
    rounds: [1]
    status: resolved
  - key: d11-launch-time-events-not-in-logs
    rounds: [2]
    status: resolved
  - key: d13-verify-line-misses-kept-covers
    rounds: [2]
    status: resolved
  - key: d8-pause-after-on-skipped-row-undefined
    rounds: [2]
    status: resolved
  - key: tag-rename-blinds-tmux-llm-guards
    rounds: [2]
    status: resolved
  - key: d9-build-time-key-refusal-has-no-source
    rounds: [2]
    status: resolved
  - key: d14-tree-scope-emptying-not-idempotent
    rounds: [2]
    status: resolved
  - key: one-tag-scan-cannot-land-with-first-retag
    rounds: [2]
    status: resolved
  - key: d4-guard-step-green-entry-silent
    rounds: [2]
    status: resolved
  - key: stale-entered-mark-after-skipped-route
    rounds: [2]
    status: resolved
  - key: d9-numeric-switch-lets-through
    rounds: [2]
    status: resolved
  - key: d5-testtiming-exposure-misstated
    rounds: [2]
    status: resolved
  - key: d4-bound-omits-ignored-files
    rounds: [2]
    status: resolved
  - key: d3-rationale-names-wrong-recorded-pass
    rounds: [2]
    status: resolved
  - key: requirement-sits-in-notes-landing-kind-map
    rounds: [2]
    status: resolved
---

Round 1 raised 14 findings and round 2 another 14, with a BLOCKING one on D11's launch-time source.
Round 3 raised none: the reviewer checked D11, D13, D8, D14, the gate-only tag guards and D7 against the code and found each sound.
No key recurs, none is open, and every earlier key is recorded resolved.
