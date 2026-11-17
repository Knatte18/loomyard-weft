---
round: 1
ledger:
  - key: code-scope-writer-claimed-decision-source
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dotnet-hub-review-code-fan-omits-code-scope
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: lens-phrase-row-asserts-static-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
The code-scope lens traces changes to the record's own `D<n>` headings, which the rubric bars as a measuring stick when a decision record alone is the answer key.
The dotnet-hub doc's review-code fan example omits the new lens.
The new lens-phrase test row asserts static prose.
Each fix text names the exact edit, so all three are mechanical.
