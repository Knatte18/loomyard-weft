---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d13-verify-line-misses-kept-covers
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-pause-after-on-skipped-row-undefined
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tag-rename-blinds-tmux-llm-guards
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-build-time-key-refusal-has-no-source
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d14-tree-scope-emptying-not-idempotent
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: one-tag-scan-cannot-land-with-first-retag
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-guard-step-green-entry-silent
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: stale-entered-mark-after-skipped-route
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d9-numeric-switch-lets-through
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d5-testtiming-exposure-misstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-bound-omits-ignored-files
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-rationale-names-wrong-recorded-pass
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: requirement-sits-in-notes-landing-kind-map
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 14 findings; none reappears in round 2, so all are recorded resolved.
Round 2 raised 14 new findings, all open.
The BLOCKING one is that D11 reads per-package launch time from start events the gate logs never hold, which also falsifies the OSL-1046 before/after grounding.
Seven MEDIUM findings cover the kept `covers` hole in the retag Verify line, `pause_after` on a skipped row, the old `tmux` and `llm` guards, D9's build-time key source, D14's non-idempotent `tree_scope` emptying, the one-tag scan's landing order and the guard step's green entry.
The six LOW findings are bound, rationale and relocation corrections.
