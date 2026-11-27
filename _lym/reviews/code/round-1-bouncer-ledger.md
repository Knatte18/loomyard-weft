---
round: 1
ledger:
  - key: validate-description-refusal-literal-key-name
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: describe-review-seat-short-key-name
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: forbidden-terms-refusal-no-way-forward-or-test
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: describe-reviewer-not-behind-mechanical-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: describe-reviewer-budget-runs-from-row-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-danger-door-and-blast-radius
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: describe-entry-gate-order-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: forbidden-terms-forwarding-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: code-review-rubric-no-tooling-name-check
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: held-turn-reminder-for-operator-turn-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: tooling-name-rule-missing-from-writer-stencils
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: describe-review-findings-resent-after-reattach
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pr-review-round-ends-with-unsynced-text
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: final-summary-spec-sentence-under-wrong-heading
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-describe-review-load-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: set-recorded-text-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: describe-review-gate-error-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: describe-review-attempts-comment-reattach-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: chair-timeout-stuck-drops-gate-attempts-and-spans
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: forbidden-pattern-matching-empty-passes-lines
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: wire-lightweight-comment-no-config-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so every entry is new.
The open entries cluster in four areas:
the description reviewer's timing and hand-off, the forbidden-term key's messages and coverage,
the rule that repository text names nothing of the tooling, and keeping the pull request's text in step.
