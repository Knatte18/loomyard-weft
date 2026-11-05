---
round: 1
ledger:
  - key: raddle-file-half-done-not-resumed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overview-raddle-and-usage-dirs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: raddle-file-done-marker-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: raddle-approve-ignores-waiting-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: record-write-resume-wrong-support-log
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
The gating one is the half-done `FileRaddle` filing: a crash between the board save and the store write leaves filed entries with no done marker, and a retry bounces the proposal as slug-taken.
Its fix adds an `AlreadyFiled` seam across `boardengine`, `loomshed`, `shedrecipe` and `loomcli`, so it changes an interface and needs a fresh review.
The other four are local corrections the finding text already fixes: the stale overview rows, the missing tests for the done marker and `StoreDone`, the `raddle-approve` waiting-notice check, and the support log choice on a resumed Record-Write.
