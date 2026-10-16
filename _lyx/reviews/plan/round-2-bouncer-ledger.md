---
round: 2
ledger:
  - key: restored-prompt-clear-undercounts-blank-rows
    rounds: [1]
    status: resolved
  - key: styled-capture-trailing-cells-differ-from-plain
    rounds: [1]
    status: resolved
  - key: daemon-log-open-failure-wording
    rounds: [1]
    status: resolved
  - key: interrupt-restoring-untracked-guid-refusal-unreached
    rounds: [1]
    status: resolved
  - key: sgr-faint-reads-extended-colour-subparameters
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: daemon-log-unreadable-branch-unreached
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: daemon-log-readonly-row-needs-root-windows-skip
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plain-capture-trim-is-ascii-spaces-not-whitespace
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's four keys are resolved: round 2's review re-checked their sites and rejected them as no longer defects.
Round 2 raised four new findings.
The gating one is the unreached "opens for append but cannot be read" branch in card 12; the others are the SGR extended-colour sub-parameter rule in card 2, the root and Windows skip for card 12's 0o444 row, and the ASCII-space versus whitespace trim wording in card 2.
All four fixes are local text and row additions the findings already spell out.
