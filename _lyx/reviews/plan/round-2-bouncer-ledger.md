---
round: 2
ledger:
  - key: card2-outside-root-exemption
    rounds: [1]
    status: resolved
  - key: card2-scan-end-branches-unreached
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: card4-read-error-after-spent-restep
    rounds: [1]
    status: resolved
  - key: card2-bound-omits-scan-end-passes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings; round 2 raised two LOW ones.
The outside-root exemption and the read-error-after-re-step findings were not raised again.
The scan-end rows finding narrowed from a design-class MEDIUM to a scope-class LOW about the unplaceable `-C` row alone.
The new consistency LOW asks card 2's Bound paragraph to list every scan-end pass its Intent gives.
