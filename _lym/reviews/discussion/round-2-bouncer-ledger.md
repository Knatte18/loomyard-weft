---
round: 2
ledger:
  - key: split-perch-auto-accept-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-admission-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-status-skip-never-ends
    rounds: [1]
    status: resolved
  - key: d10-family-premise-false
    rounds: [1]
    status: resolved
  - key: d8-release-placement
    rounds: [1]
    status: resolved
  - key: perch-raddle-write-end-unstated
    rounds: [1]
    status: resolved
  - key: d7-after-filing-no-recorder
    rounds: [1]
    status: resolved
  - key: d7-refusal-changes-raddle-runs
    rounds: [1]
    status: resolved
  - key: d10-refusal-returns-to-resplit
    rounds: [1]
    status: resolved
  - key: open-risk-writer-choice
    rounds: [1]
    status: resolved
  - key: filing-mark-after-board-write
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: rearm-keeps-generation-record
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-raddle-write-no-rearm
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: motif-open-risk-outdated
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d6-rejected-alternative-in-record
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-record-write-reason-twice
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's ten keys: two reopen in round 2 (the auto-accept route, now also through the soft-cap rule and at BLOCKING, and the D6 admission's unstated bound).
The other eight are resolved, since the round 2 review raised nothing on them and rejected its own focus questions on D7, D8, D10 and perch-mode Raddle-Write.
Round 2 adds six keys: the filing-mark write order (BLOCKING), the re-arm's generation record, the goto into Raddle-Write, an outdated Open risk, a rejected alternative argued in D6, and a duplicated reason.
