---
round: 3
ledger:
  - key: d4-single-invocation-premise-false-for-recovery-strand
    rounds: [1]
    status: resolved
  - key: misspelled-skill-open-risk-contradicts-shuttle-skip
    rounds: [1]
    status: resolved
  - key: d7-any-skill-allowed-without-stated-bound
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: criterion-leaves-discussion-plan-fix-rows-unpinned
    rounds: [1]
    status: resolved
  - key: sandbox-coverage-non-applicability-line-in-constraints
    rounds: [1]
    status: resolved
  - key: d4-names-landingshed-as-loading-constructor
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, round 2 two, round 3 two.
Four round 1 keys stay resolved.

D7's bound was reopened as a MEDIUM design finding in round 2.
Round 3 confirms the bound is stated and the examples hold, and finds only wording inexactness in how it describes the deny (subtree and `all` shapes, tier clause scope, the third variable shape).
The finding text fixes the replacement wording, so the key stays open at LOW consistency, mechanical.

The D4 key carried the round 2 NIT about the loading constructor.
Round 3 widens it to the load site and the once-per-process phrasing, at LOW consistency, with the replacement sentences given in the finding.
