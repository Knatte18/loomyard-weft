---
round: 1
ledger:
  - key: stored-resume-line-keeps-remote-control
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: summary-misstates-absent-orch-yaml
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-row-registration-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three non-gating findings.
F1 concerns reed replaying a stored resume line that carries remote control; it is deferred to the parent.
F2 corrects the summary's claim about a hub with no `orch.yaml`.
F3 asks for a test pinning the orch row's `Reinterpretations` entry.
