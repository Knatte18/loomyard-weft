---
round: 1
ledger:
  - key: invocable-counts-persistent-flags
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: fabric-clone-long-over-cap
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: watcher-index-binary-decision-uncarried
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: duplicate-audience-assertions-cards-6-8
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: runnable-group-print-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review; there is no prior ledger.
Open: the persistent-flag `Invocable` defect (F1) and the unmeasured `Long` trim lists (F2) are BLOCKING design findings.
The uncarried Watcher index binary decision (F3), the duplicated audience assertions (F4) and the runnable-group print rule (F5) are lesser.
