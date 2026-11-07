---
round: 1
ledger:
  - key: stale-mail-line-untested-under-failing-outbox-write
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: savefault-errreadbudget-unsaved-reset-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-carries-old-outbound-guarantee
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: guarantee-stated-without-restart-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
The one gating finding is the missing stale-mail test under a failing outbox write.
The other three are a LOW test gap, a stale line in `docs/overview.md`, and a guarantee missing its restart bound.
Each fix is a local correction the finding text already spells out.
