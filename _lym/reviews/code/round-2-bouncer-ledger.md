---
round: 2
ledger:
    - key: validate-description-refusal-literal-key-name
      rounds:
        - 1
      status: resolved
    - key: describe-review-seat-short-key-name
      rounds:
        - 1
      status: resolved
    - key: forbidden-terms-refusal-no-way-forward-or-test
      rounds:
        - 1
      status: resolved
    - key: describe-reviewer-not-behind-mechanical-gate
      rounds:
        - 1
      status: resolved
    - key: describe-reviewer-budget-runs-from-row-start
      rounds:
        - 1
        - 2
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: merge-danger-door-and-blast-radius
      rounds:
        - 1
      status: resolved
    - key: describe-entry-gate-order-untested
      rounds:
        - 1
      status: resolved
    - key: forbidden-terms-forwarding-untested
      rounds:
        - 1
        - 2
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: code-review-rubric-no-tooling-name-check
      rounds:
        - 1
      status: resolved
    - key: held-turn-reminder-for-operator-turn-missing
      rounds:
        - 1
      status: resolved
    - key: tooling-name-rule-missing-from-writer-stencils
      rounds:
        - 1
        - 2
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: describe-review-findings-resent-after-reattach
      rounds:
        - 1
      status: resolved
    - key: pr-review-round-ends-with-unsynced-text
      rounds:
        - 1
      status: resolved
    - key: final-summary-spec-sentence-under-wrong-heading
      rounds:
        - 1
      status: resolved
    - key: empty-describe-review-load-untested
      rounds:
        - 1
      status: resolved
    - key: set-recorded-text-failure-paths-untested
      rounds:
        - 1
      status: resolved
    - key: describe-review-gate-error-paths-untested
      rounds:
        - 1
      status: resolved
    - key: describe-review-attempts-comment-reattach-claim
      rounds:
        - 1
      status: resolved
    - key: chair-timeout-stuck-drops-gate-attempts-and-spans
      rounds:
        - 1
      status: resolved
    - key: forbidden-pattern-matching-empty-passes-lines
      rounds:
        - 1
      status: resolved
    - key: wire-lightweight-comment-no-config-claim
      rounds:
        - 1
      status: resolved
    - key: review-timeout-warn-literal-key-name
      rounds:
        - 2
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: stale-pr-text-hold-no-way-forward
      rounds:
        - 2
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
---
Round 2 resolved most of round 1's entries.
Three stay open: the description reviewer's time budget, the untested forwarding of the forbidden terms
(now narrowed to the beam recorders), and the tooling-name rule (now narrowed to the beam describer's stencil).

The two new entries are both BLOCKING.
One is a key name written as a literal in the review's timeout warning.
The other is the hold on a pull request review whose text cannot be built, which names no way forward;
it follows from the round-1 fix for a review round ending with unsynced text.
