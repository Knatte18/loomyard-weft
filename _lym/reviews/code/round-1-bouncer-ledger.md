---
round: 1
ledger:
  - key: pr-review-does-not-wait-on-orch-answer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failed-ask-record-write-resends-request
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parent-review-wiring-closures-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all class `design`.
The MEDIUM one is that PR-Review mails the orch once and then ends on the operator's ending without waiting for the orch's answer.
Two LOW findings follow: a failed `ParentReviewAsked` write after a successful post re-sends the ask, and the `ParentReviewWanted` and `AskParent` closures in the loom wiring have no test.
No earlier ledger exists.
