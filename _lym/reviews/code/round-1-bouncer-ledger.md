---
round: 1
ledger:
  - key: card-watch-skip-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: guard-step-watch-drop-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: armdeadline-package-var-clock
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: production-deadline-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancellation-test-child-not-running
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-pattern-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusal-double-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, all open.
F1 and F2 are untested watch behaviors in `internal/gatecli`.
F3 is the `armDeadline` package var against `PATTERN-test-speed`.
F4 to F7 are LOW: a lost production-deadline test, a racy cancellation test, a stale guard section in the pattern background,
and a doubled way-forward clause in the `GuardMisplaced` refusal.
