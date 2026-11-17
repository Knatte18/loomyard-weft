---
round: 1
ledger:
  - key: rc-button-drawn-without-button-dir
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rc-on-verb-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rc-button-keyed-on-strand-not-strand-name
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: paneoptions-docs-miss-rc-click-read
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-template-rc-comments-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: slack-bar-dir-help-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failed-launch-mark-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: duplicate-set-option-assertion
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, F1 to F8, in the order of the ledger above.
F1 is the only gating one: the RC button's draw condition does not require the button directory, so a click in a session without one fails before the verb runs.
F3 shares F1's site and fix.
The rest are test-coverage, test-economy and stale-comment findings.
