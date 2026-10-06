---
round: 1
ledger:
  - key: d4-already-landed-empty-diff
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-local-weft-vs-archive-tag-precedence
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-bound-contradicts-r3
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-skip-options-and-originless-repo
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-guard-bypass-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d6-fast-forward-rollback
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: added-section-mixes-scheduling
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 opened eight findings.
The blocking one is D4: the already-landed path reads an empty diff after a non-squash landing or a catch-up merge-in, so the notice is not queued and R2's recovery claim is false.
D6 carries three open items: the local-weft versus archive-tag precedence, the Bound that R3 contradicts, and the fast-forward rollback.
D5's skip options and origin-less repos, D7's unbounded guard bypass, four missing acceptance assertions and the mixed "Added after Discussion" section remain open.
