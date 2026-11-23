---
round: 1
ledger:
  - key: layout-leaves-changed-lines-over-limit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: long-line-report-covers-wrong-lines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: long-line-report-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: latest-fitting-break-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: evidence-lacks-base-figures
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none relabeled.
F1 (`layout-leaves-changed-lines-over-limit`) is the blocker: `layoutParagraph` can leave changed lines at or past the limit, so the tool adds line-length findings.
F2 (`long-line-report-covers-wrong-lines`) changes how the hand-rewrite report picks its lines, and its fix adds a new return from `collectEdits`.
F3 and F4 are coverage gaps with literal-row fixes.
F5 is an evidence gap in the change description, to be updated after F1's fix.
