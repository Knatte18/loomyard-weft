---
round: 1
ledger:
  - key: codex-acceptance-criterion-unmet
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: uncommitted-log-figures-unmarked
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: median-stated-as-upper-middle
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-mail-hook-cost-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unindented-continuation-line
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: lines-with-two-independent-clauses
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings on the report: one MEDIUM decision-class gap (Codex hook behavior untested, deferred to the parent), three LOW and two NIT consistency fixes.
None is gating, so the loop converges.
