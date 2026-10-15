---
round: 1
ledger:
  - key: cli-go-package-doc-scoped-to-one-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-state-cache-race-guard-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-line-test-expectations-underspecified
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none at MEDIUM or worse.
F1 widens card 2's package-doc edit in `internal/battencli/cli.go`, F2 gives the `PeekState` not-found branch in `orchStateCache.Poll` a reaching test, and F3 pins three test expectations.
