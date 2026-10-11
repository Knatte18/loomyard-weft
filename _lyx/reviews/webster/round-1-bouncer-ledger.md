---
round: 1
ledger:
  - key: publish-retry-tests-serial
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: pause-detected-by-reason-prefix
    rounds: [1]
    status: open
    class: design
    severity: NIT
---

Round 1 raised two findings.
F1 (`publish-retry-tests-serial`) is four `internal/battenshed` publish-retry top-level tests that skip `t.Parallel` without touching process-global state.
F2 (`pause-detected-by-reason-prefix`) is `stop` in `internal/battenshed/land.go` reading the pause rule back out of a reason string prefix.
Both are open until a fresh round confirms them.
