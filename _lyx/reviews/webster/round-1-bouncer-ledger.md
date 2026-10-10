---
round: 1
ledger:
  - key: rebaseline-refusals-untested
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: edited-done-cards-duplicate-gate-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: ungated-waiting-empty-list-not-at-once
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: loop-agrees-stale-waited-out-comment
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised four findings.
F1 (BLOCKING) is the missing test for the reworded post-rebaseline zero-batch refusal and quarry transient in `internal/websterengine/runlevel.go`.
F2 is a duplicated gate row in `TestEditedDoneCards`.
F3 is `expiredTurnEnd` holding an ungated waiting turn end with an empty task list.
F4 is a stale comment on `loopAgreesWithState`.
