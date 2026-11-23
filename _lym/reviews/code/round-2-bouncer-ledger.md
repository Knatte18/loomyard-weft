---
round: 2
ledger:
  - key: decision-escalation-accept-bypasses-parent-on-blocking
    rounds: [1]
    status: resolved
  - key: code-auto-accept-meaning-change-no-reinterpretation
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: consolidation-drops-origin-focus
    rounds: [1]
    status: resolved
  - key: focus-exemption-ungated-in-round-context
    rounds: [1]
    status: resolved
  - key: review-step-changed-hunk-rule-lacks-focus-exception
    rounds: [1]
    status: resolved
  - key: below-budget-blocking-row-hits-rejudge
    rounds: [1]
    status: resolved
  - key: unreadable-ledger-branch-untested
    rounds: [1]
    status: resolved
  - key: hard-cap-only-window-cause-unasserted
    rounds: [1]
    status: resolved
  - key: stale-code-auto-accept-descriptions
    rounds: [1]
    status: resolved
  - key: switch-test-needs-keep-marker
    rounds: [1]
    status: resolved
  - key: below-window-decision-accept-passes-open-blocking
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hardener-focus-permission-wiring-untested
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: budget-escalation-decision-detail-unasserted
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1's ten keys: eight are resolved by the round-1 fixes, and the first (`decision-escalation-accept-bypasses-parent-on-blocking`) is resolved for the soft-cap window but its below-window variant is the new F13 key.
`code-auto-accept-meaning-change-no-reinterpretation` is carried as F2: the reinterpretation misses hubs that take the template default and absent `loom.yaml`, and the summary's door is still two-way.
Round 2 adds three BLOCKING design findings: F13 (an arbiter accept below the soft-cap window passes open non-decision BLOCKING entries), F11 (the hardener focus permission's two wiring sites are untested), and F12 (the budget escalation's decision-finding detail is unasserted).
