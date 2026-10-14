---
round: 1
ledger:
  - key: card2-redundant-test-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: unreadable-dir-rows-missing-go-mod
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unreadable-test-file-scan-error-vs-refusal
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round; there is no prior ledger.
F1 maps to `card2-redundant-test-rows`: card 2 plans test rows that existing rows already cover.
F2 maps to `unreadable-dir-rows-missing-go-mod`: the unreadable-directory rows name no root `go.mod`, so card 5's early return keeps the refusal from being reached.
F3 maps to `unreadable-test-file-scan-error-vs-refusal`: no card settles whether an unreadable `_test.go` file is a scan-error prompt line or D3's read-error refusal.
All three are LOW and non-gating.
