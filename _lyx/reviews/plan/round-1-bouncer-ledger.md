---
round: 1
ledger:
  - key: tracked-run-dir-seed-without-status
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cleanup-prime-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: testtiming-resign-arrow-missing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: afterrun-caller-miscount
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: board-segment-duplicates-openhub
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: log-segment-bounds-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 was the first review; no earlier ledger exists.
Six findings, none gating: F1 maps to `tracked-run-dir-seed-without-status`, F2 to `cleanup-prime-refusal-untested`, F3 to `testtiming-resign-arrow-missing`, F4 to `afterrun-caller-miscount`, F5 to `board-segment-duplicates-openhub`, F6 to `log-segment-bounds-untested`.
