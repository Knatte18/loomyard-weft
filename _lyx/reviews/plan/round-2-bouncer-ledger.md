---
round: 2
ledger:
  - key: card21-misses-marker-integration-test
    rounds: [1]
    status: resolved
  - key: tackler-model-spec-refusal-no-spec-row
    rounds: [1]
    status: resolved
  - key: clean-tree-check-drops-untracked-paths
    rounds: [1]
    status: resolved
  - key: status-derived-stop-no-envelope
    rounds: [1]
    status: resolved
  - key: card15-verb-bound-wrong
    rounds: [1]
    status: resolved
  - key: istacklerstrand-no-consumer
    rounds: [1]
    status: resolved
  - key: resume-second-seed-driver-reader-no-pattern-edit
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: friction-failed-done-names-unwritten-report
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: binary-restep-marker-blocks-retry
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: recheck-preempts-busy-row
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: validate-local-values-negative-knob-no-test
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: verify-record-closure-carries-only-target
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: spawn-settle-record-without-agent
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings; round 2 re-raised none of them, so all six are resolved.
Round 2 raised seven new findings, F1 to F7, in the order of the round 2 keys above.
F1 is BLOCKING and cites `PATTERN-driver-choice-single-site`; F2 and F3 are MEDIUM; F4 to F7 are LOW.
