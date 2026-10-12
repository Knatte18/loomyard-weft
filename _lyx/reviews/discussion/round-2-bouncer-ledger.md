---
round: 2
ledger:
  - key: d4-same-turn-read-of-uncovered-files
    rounds: [1]
    status: resolved
  - key: step1-offset-read-no-edited-file-carveout
    rounds: [1]
    status: resolved
  - key: d3-step1-remaining-sentences-undecided
    rounds: [1]
    status: resolved
  - key: d1-quarry-unavailable-handle-reason
    rounds: [1]
    status: resolved
  - key: d2-named-none-resolved-rendering
    rounds: [1]
    status: resolved
  - key: support-log-grouped-spec-premise
    rounds: [1]
    status: resolved
  - key: scope-out-measurement-deliberation
    rounds: [1]
    status: resolved
  - key: d4-unresolved-member-read-turn-pin
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: open-risk-false-premise-resigned-member
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised seven findings; round 2's review raises none of those keys, so all seven are resolved.
Round 2 raised two new findings.
F1 (`d4-unresolved-member-read-turn-pin`): D4 pins a whole-file read of an unresolved member glyph's file to the turn after the card read, before the file is located.
F2 (`open-risk-false-premise-resigned-member`): the Open risk rests on a false premise, and the real gap is a member re-signed by an earlier card of the batch.
