---
round: 1
ledger:
  - key: refuse-raw-comment-compares-other-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shellguard-doc-overstates-steer-rule
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two non-gating consistency findings in `internal/shellguard`.
F1 is a comment in `refuseRawRow` that explains its notice by comparison to another row.
F2 is a package doc sentence claiming every row's steer carries the full rule.
Both fixes are local rewordings the review text already spells out.
