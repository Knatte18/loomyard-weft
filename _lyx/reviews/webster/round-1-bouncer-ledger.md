---
round: 1
ledger:
  - key: carry-over-error-paths-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: resume-long-table-misaligned
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: fence-h2-scan-triplicated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so there is no prior ledger.
F1 maps to `carry-over-error-paths-untested`: the anchor-relative path error and the unreadable-ledger error in `buildCarryOver` have no test, and the refusal-spec row omits the path cause.
F2 maps to `resume-long-table-misaligned`, a one-space padding slip in the resume `Long` table.
F3 maps to `fence-h2-scan-triplicated`, the fence and H2 scan copied into three functions of `discussionparser`.
None is gating.
