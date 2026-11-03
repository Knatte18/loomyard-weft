---
round: 1
ledger:
  - key: ask-path-no-real-tmux-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: ask-comma-in-filled-placeholder
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ask-before-confirm-order-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: ask-strand-beam-render-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: reedengine-doc-stale-entry-description
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reedcli-test-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no prior ledger.
Six findings, none gating: the MEDIUM is a scope finding about a missing real-tmux ask test (F1), the single design finding is LOW (F2, comma in a filled placeholder value), and the rest are test-coverage and doc-consistency items (F3 to F6).
Each fix text names the exact edit, so all are mechanical.
