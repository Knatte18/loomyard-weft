---
round: 1
ledger:
  - key: top-rule-label-without-glyph-or-ellipsis
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: held-start-notice-lacks-report-opening
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: interactive-guard-of-held-start-notice-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: held-start-notice-delimited-reason-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: send-start-prompt-doc-overstates-notice
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), none gating.
F1 is a MEDIUM scope finding on `isTopBoxRule`, F2 and F5 are consistency, and F3 and F4 are LOW design findings that ask for test rows.
Each fix is local and its text is fixed by the finding.
