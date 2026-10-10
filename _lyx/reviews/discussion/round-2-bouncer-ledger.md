---
round: 2
ledger:
  - key: d1-bound-false-premise-budget-exempt-stuck
    rounds: [1]
    status: resolved
  - key: d4-turn-start-sourced-from-shadow-after-hold
    rounds: [1]
    status: resolved
  - key: d4-bound-omits-provider-without-turn-start-line
    rounds: [1]
    status: resolved
  - key: d3-wall-delta-monotonic-reading-not-stripped
    rounds: [1]
    status: resolved
  - key: d3-subtraction-input-mismatch-apierror-fallback
    rounds: [1]
    status: resolved
  - key: d5-display-only-bound-asserted-not-shown
    rounds: [1]
    status: resolved
  - key: d3-notice-text-keeps-reporting-wording
    rounds: [1]
    status: resolved
  - key: d3-stamp-change-drop-versus-add-order-open
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: budgetexempt-constraint-cites-ungoverning-patterns
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings, two BLOCKING design findings among them.
Round 2's review did not re-raise any of them, and its focus verdicts found D1, D4, D5 and the D3 wall delta sound, so all seven are resolved.

Round 2 raised two new findings.
F1 asks D3 to state that a check seeing the activity stamp change drops the accumulation without adding its own gap, and AC9 to pin that order.
F2 asks the BudgetExempt constraint to drop its two pattern citations and state the rule on its own terms.
Both are non-gating.
