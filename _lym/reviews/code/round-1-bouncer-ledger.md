---
round: 1
ledger:
  - key: prreview-empty-credit-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prreview-engines-run-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: landingmessage-trailers-naming
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prcontinue-header-trailer-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
Two are test-coverage gaps in PR-Review (empty credit block, engines-run read failure), both gating.
Two are stale "trailer" wording in `LandingMessage` and the `prcontinue.go` header, both LOW.
All four fixes are local and fixed by the finding text.
