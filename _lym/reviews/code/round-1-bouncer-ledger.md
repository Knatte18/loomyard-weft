---
round: 1
ledger:
  - key: skip-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: nonfatal-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: build-comment-omits-once-only-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: poisoned-message-names-only-copyhub
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all in `internal/hubforge`.
F1 (MEDIUM design) is the skip-refusal test gap, and F2 (LOW design) is the non-fatal failure test gap.
F3 (LOW consistency) is the missing once-only-registration clause in the `Registration.Build` comment and doc.go.
F4 (NIT consistency) is the poisoned-template message naming only `CopyHub`.
Each fix is spelled out in the review, so all four are mechanical.
