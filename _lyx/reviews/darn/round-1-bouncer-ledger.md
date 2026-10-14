---
round: 1
ledger:
  - key: orch-last-line-rule-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-orch-duplicate-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 (`orch-last-line-rule-untested`): `TestCountOrch` writes each message id on one line, so nothing pins `readTranscript` taking usage and timestamp from the last line.
F2 (`summary-overstates-orch-duplicate-count`): the landing summary implies the orch counter reports dropped earlier lines, but only the run counter does.
Both fixes are local and spelled out by the finding text.
