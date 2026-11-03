---
round: 1
ledger:
  - key: status-title-departs-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: battencli-title-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: kept-strand-keeps-old-title
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: status-strand-title-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runs-strand-title-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings.
The status window title `<handle>: <recipe>` departs from the entry's `<handle>: overview` with no recorded decision (F1, deferred to the parent).
The Open-Window titled name in `commandDeps` has no test (F2, gating).
Three LOW findings cover a kept strand keeping its old title (F3) and two unpinned title wirings (F4, F5).
