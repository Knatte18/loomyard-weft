---
round: 1
ledger:
  - key: decision-escalation-accept-bypasses-parent-on-blocking
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: code-auto-accept-meaning-change-no-reinterpretation
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: consolidation-drops-origin-focus
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: focus-exemption-ungated-in-round-context
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: review-step-changed-hunk-rule-lacks-focus-exception
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: below-budget-blocking-row-hits-rejudge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unreadable-ledger-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hard-cap-only-window-cause-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-code-auto-accept-descriptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: switch-test-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings against a first-round diff with no earlier ledger.
Two BLOCKING design findings lead: an arbiter accept can pass open BLOCKING findings without the parent, and `code_auto_accept` changes meaning without a reinterpretation.
Three more design findings are gating: lost `origin: focus` in consolidation, an ungated focus exemption, and a test row on the wrong path.
The rest are consistency and test-coverage items.
