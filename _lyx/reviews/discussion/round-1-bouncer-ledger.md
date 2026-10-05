---
round: 1
ledger:
  - key: d5-package-var-time-seam
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-retag-drops-gates
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-llm-decision-date
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-coverage-blind-to-spawned-binaries
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-socket-removal-race
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d7-llmkit-exemption-unenforced
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d8-background-file-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: constraints-risks-restatement
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eight findings, none carried from earlier rounds.
The gating ones are the package-var time seam (F1), the retag that leaves tests under a tag no gate runs (F2) and coverage blindness to spawned binaries (F4).
The rest are an unenforced llmkit exemption bound, a socket removal window, a wrong date, a wrong file count and restatement between sections.
