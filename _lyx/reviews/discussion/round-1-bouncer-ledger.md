---
round: 1
ledger:
  - key: d2-keep-directive-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-splice-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-git-hook-module-code
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-untagged-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-missing-boundary-scenarios
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: rationale-tallies-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-wiring-condition-permissive
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d4-stop-condition-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: docs-bullet-omits-tool-doc
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised nine findings, all open.
The two gating ones are unstated bounds: D2's keep directive (misplaced, duplicated, stale, covering-pool membership) and D3's splice (tag set mismatch, section removal on a failed package, missing out file).
The rest are a D1 hook gap, an overstated acceptance bullet, missing boundary scenarios, tallies in Rationale (one disagreeing with the support log), a permissive D6 wiring sentence, a doubled D4 stop wording and an omitted package doc.
