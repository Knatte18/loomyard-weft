---
round: 1
ledger:
  - key: click-line-cd-outside-output-discard
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: click-line-no-pwsh-test-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 (first round, no prior ledger) raised two findings.
The directory change in the pinned click line and the confirmed line is not wrapped in the output discard, so a removed run directory writes to stderr and opens a tmux view-mode pane.
The pinned click line also has no pwsh test row and no direct `ExitSuccess` assertion.
Both fixes are local corrections whose text the findings already fix.
