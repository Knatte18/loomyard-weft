---
round: 1
ledger:
  - key: card23-misses-battencli-step-tests
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card19-misses-orch-refusalcase
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: promote-start-guard-misses-stop-after-step2
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card9-order-beam-refusal-unreached
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card23-extends-rewritten-cli-file
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card24-beam-reed-session-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: missing-origin-record-refusal-wrong-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: adopt-local-records-branch-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card24-window-help-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: promote-root-pair-dead-end
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: card4-parent-refusals-weak-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: slug-rule-suggestion-unchecked
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-false-refusal-names-wrong-verbs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card23-self-pair-state-misstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card34-cliwire-constraint-silent
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Fifteen findings are open, none BLOCKING.
Two are gating: `promote-start-guard-misses-stop-after-step2` (F3) and `card24-beam-reed-session-unbounded` (F6), both `design` at MEDIUM.
The rest are `consistency` findings on stale assertions and texts, refusals without a way forward, and one `decision` finding on the cliwire constraint.
