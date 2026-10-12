---
round: 1
ledger:
  - key: d4-same-turn-read-of-uncovered-files
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: step1-offset-read-no-edited-file-carveout
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-step1-remaining-sentences-undecided
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d1-quarry-unavailable-handle-reason
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-named-none-resolved-rendering
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: support-log-grouped-spec-premise
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: scope-out-measurement-deliberation
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings; none had an earlier ledger.
The two gating ones concern D4 and D3 of `decision-record.md`: an impossible same-turn read of files only the card file names, and an offset read that ignores files already edited in the batch.
The other five are LOW wording and placement fixes.
