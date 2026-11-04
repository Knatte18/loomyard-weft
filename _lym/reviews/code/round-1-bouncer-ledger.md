---
round: 1
ledger:
  - key: bash-early-return-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: git-reads-write-options
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: matcher-test-misses-bash
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: max-nudges-comment-session-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lookup-table-rows-missing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: agent-id-guard-one-separator
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: agent-nudged-message-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: agent-sidechain-switch-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: help-text-still-says-session
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings: one MEDIUM, seven LOW and one NIT.
The only gating finding is the untested early return for a non-lookup Bash call.
Most others are test-coverage gaps or wording the diff left behind.
Every fix is a local correction whose result the finding text already fixes.
