---
round: 3
ledger:
  - key: split-perch-auto-accept-unbounded
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-admission-unbounded
    rounds: [1, 2, 3]
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
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rearm-keeps-generation-record
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-raddle-write-no-rearm
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: motif-open-risk-outdated
    rounds: [2]
    status: resolved
  - key: d6-rejected-alternative-in-record
    rounds: [2]
    status: resolved
  - key: no-record-write-reason-twice
    rounds: [2]
    status: resolved
  - key: d6-order-check-underdetermined
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

The auto-accept route and the D6 admission bound have been open since round 1 and are again open at MEDIUM: the soft-cap bound understates what the rule accepts, and the admission bound claims knowledge the lock lacks.
The filing-mark ordering key dropped from BLOCKING to a LOW residual, the store-done retry case.
The goto re-arm key returns at BLOCKING because `freshRaddleEntry` never matches a goto entry as `shedengine` writes it; the re-arm generation-record key returns at MEDIUM for a perch that had not settled.
The three round 2 consistency keys raised nothing in round 3 and are resolved.
Round 3 adds one key, the D6 order check, at LOW.
