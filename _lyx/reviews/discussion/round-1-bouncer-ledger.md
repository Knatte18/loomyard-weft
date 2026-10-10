---
round: 1
ledger:
  - key: d6-dead-fork-recover-batch-refuses-at-cap
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-quoted-glob-vs-fabric-diff-acceptance
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-json-pin-no-acceptance-row-wrong-home
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-glob-bound-assumes-bash-parens-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-after-dashdash-premise-false-for-ls-remote
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d5-webstercli-missing-from-scope
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: open-risks-non-risk-and-routine-note
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: d4-miscounts-rows-already-present
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the discussion pair, so there is no earlier ledger.
It raised nine findings.
F1 (D6 dead-fork case), F3 (D3 pin) and F4 (D2 shell dialect) gate.
The rest are non-gating and open for the fixer.
