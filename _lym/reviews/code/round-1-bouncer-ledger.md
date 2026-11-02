---
round: 1
ledger:
  - key: note-key-list-collapses-to-count
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: flush-before-cobra-error-and-local-envelope-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: collapse-boundary-and-note-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cli-cobra-background-omits-envelope-flag
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: second-terminal-writer-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none from an earlier round.
The one gating finding is the note-key list that collapses to a count (F1).
The other four are test gaps (F2, F3) and consistency items (F4, F5), each with a local fix its text already fixes.
