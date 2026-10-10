---
round: 2
ledger:
  - key: rebaseline-refusals-untested
    rounds: [1]
    status: resolved
  - key: edited-done-cards-duplicate-gate-row
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: ungated-waiting-empty-list-not-at-once
    rounds: [1]
    status: resolved
  - key: loop-agrees-stale-waited-out-comment
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: touched-files-not-gofmt-clean
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: waiting-done-row-duplicates-background-table
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1's BLOCKING missing-test finding and the empty-list waiting finding are resolved; round 2's focus verdicts confirm both behaviors are pinned.
The `loopAgreesWithState` comment finding recurs and has grown to MEDIUM: the held-beside-busy allowance has no producer, and `doc.go` still describes the retired shell expiry.
The `TestEditedDoneCards` duplicate-assertion finding recurs at LOW.
Round 2 adds two LOW findings: three touched files that are not gofmt-clean, and a `TestPollEventsTick_Waiting` row that duplicates the background-work table.
