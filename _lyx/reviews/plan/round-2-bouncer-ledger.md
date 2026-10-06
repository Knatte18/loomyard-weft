---
round: 2
ledger:
  - key: skill-loading-decision-uncarried
    rounds: [1]
    status: resolved
  - key: card-targets-narrower-than-intent
    rounds: [1]
    status: resolved
  - key: hubforge-constraint-not-in-untag-procedure
    rounds: [1]
    status: resolved
  - key: card2-os-args-wider-than-record
    rounds: [1]
    status: resolved
  - key: card3-kept-test-covers-assertion-open
    rounds: [1]
    status: resolved
  - key: card9-hubforge-premise-wrong
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card17-outside-skill-loading-protection
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: testkit-writes-absent-from-card-targets
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 raised five findings, none `design`.
Round 2's review does not raise any of them again, so all five are resolved.

Round 2 raised three MEDIUM findings, none `design` and none BLOCKING.
F1 is card 9's wrong premise that webster's tests build hubs through `hubforge`.
F2 is card 17 sitting outside the skill-loading and stub-helper protections.
F3 is testkit writes that the untag and prune procedures license but no card's Targets name.
F2 is a new gap in the same area as round 1's skill-loading finding, not a reopening of it, and F3 is a new gap in the same area as round 1's Targets finding.
