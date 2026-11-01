---
round: 1
ledger:
  - key: crucible-smoke-commands-tags-only
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tmuxallowed-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: require-parse-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testtiming-setenv-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: require-reading-paths-fixtures
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: battencli-tier0-push-refusal-lost
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-tag-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: main-pass-counts-missing
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1, first ledger.
One gating finding: `tmuxallowed-unpinned` (design, MEDIUM).
The crucible smoke commands finding is MEDIUM scope and non-gating.
The rest are LOW test-coverage, comment and summary corrections.
