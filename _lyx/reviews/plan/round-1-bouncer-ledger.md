---
round: 1
ledger:
  - key: skill-loading-decision-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card-targets-narrower-than-intent
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: hubforge-constraint-not-in-untag-procedure
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card2-os-args-wider-than-record
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card3-kept-test-covers-assertion-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 is the first review, with no prior ledger.
It found five findings and none is `design`.
The BLOCKING one is the missing carrier for the decision record's rule that skill-loading tests in `internal/shuttleengine` stay untouched and any redundancy candidate among them is kept with a `skills-in-one-turn` reason.
The two MEDIUM findings concern card Targets narrower than their Intents and the hub-fixture constraint missing from the untag procedure.
The two LOW findings concern card 2's `os.Args` scan width and card 3's conditional test for the kept-test-still-covers criterion.
