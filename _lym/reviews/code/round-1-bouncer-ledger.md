---
round: 1
ledger:
  - key: gofmt-guard-passes-ending-only-difference
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-gofmt-walk-reach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
The gating one: `gofmtFinding` in `internal/testkit/gofmt_test.go` reports nothing when a source differs from its formatted form only by its ending.
The other: `_lym/landing/summary.md` claims the check reads every Go file of the module, while the walk skips `testdata` directories.
