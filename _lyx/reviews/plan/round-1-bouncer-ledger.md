---
round: 1
ledger:
  - key: card6-wire-origin-read-mutates
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card3-new-test-vs-extend-lint-comments-test
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card10-existing-background-files-untargeted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card10-documentation-lifecycle-skipped
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card1-second-top-level-test-false-premise
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card6-rungate-missing-resign-arrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card3-readoverview-crlf-vs-lf-blob
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card6-newverifygate-caller-count
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eight findings, all open.
The gating one is F1: card 6 wires an eager origin read that writes into the weft worktree and can fail every loom wire.
The others are test-economy and budget departures from the record (F2, F5), card 10's undeclared and mis-selected background files (F3, F4), a missing re-sign arrow (F6), a CRLF count mismatch in card 3 (F7) and a pinned caller count (F8).
