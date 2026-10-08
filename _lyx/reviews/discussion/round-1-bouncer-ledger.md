---
round: 1
ledger:
  - key: d5-nested-module-caller-coverage-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d9-fix-shape-suppresses-real-defect
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-interface-dispatch-drop-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d6-rename-of-gomod-ignored
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: requirements-only-in-open-risks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d8-go-dash-C-base-after-cd
    rounds: [1]
    status: open
    class: decision
    severity: NIT
---

Round 1 raised six findings, all open.
F1 (D5 drops caller coverage inside nested modules with no bound, contradicting D2's bound) and F2 (D9 makes a finding illegitimate when its fix is blocked) are BLOCKING design findings.
F3 to F6 are LOW or NIT: D1's unbounded interface-dispatch drop, D6's blindness to a renamed `go.mod`, two requirements living only in Open risks, and D8's unstated `go -C` base after `cd`.
