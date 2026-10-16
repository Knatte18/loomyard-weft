---
round: 1
ledger:
  - key: card2-outside-root-exemption-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: card2-scan-end-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card4-read-error-after-used-restep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 (card 2 exempts an argument outside the worktree root, which D3 does not license) maps to `card2-outside-root-exemption-unlicensed`.
F2 (card 2's `missingPackageArgs` scan-end branches lack named test inputs) maps to `card2-scan-end-branches-untested`.
F3 (card 4 leaves a status read error after a spent re-step unstated and untested) maps to `card4-read-error-after-used-restep`.
Each review fix names the exact text or rows to add, so all three are mechanical.
