---
round: 1
ledger:
  - key: rewritten-refs-envelope-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: report-empty-reason-refusal-exceeds-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
F1 asks for an assertion on the `rewritten_refs` envelope key in `TestRenumberVerbs`.
F2 asks to stop refusing a `renames` entry for an empty reason in `validateReportFields`.
Neither gates.
