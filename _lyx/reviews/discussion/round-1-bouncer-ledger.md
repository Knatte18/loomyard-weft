---
round: 1
ledger:
  - key: form-test-payload-token
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: coupling-test-optionality
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: watcher-index-stale-after-deploy
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: runnable-groups-outside-leaf-rules
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: completion-subtree-unannotated-leaf
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: help-replacement-skipped-guards
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: pointer-filter-match-rule
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: json-help-listing-decision
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: record-narrative-tally-double-statement
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings, none BLOCKING.
Three are gating design findings at MEDIUM: the coupling test compares key names only, the orch watcher renders a stale index after a deploy, and runnable groups such as `lyx config` miss the index.
The remaining findings are LOW or NIT, plus one non-gating consistency MEDIUM about the form test and the board payload token.
