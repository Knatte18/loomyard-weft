---
round: 1
ledger:
  - key: slot-wait-invisible-in-status
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: inherit-env-escape-hatch-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: test-budget-edit-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: package-docs-describe-old-behavior
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: text-only-push-line-left-to-merge
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: empty-impacted-set-tmux-pass
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised six findings, none BLOCKING.
Three are design-class MEDIUM and gate the loop: card-verify and plan-gate slot waits that status cannot see, the unbounded `InheritEnv` routing edge in card 10, and package `doc.go` files left stating replaced behavior.
The test-budget edit, the text-only push line in card 16 and the empty impacted-set `tmux` pass are non-gating.
