---
round: 1
ledger:
  - key: d2-rearm-bound-misreads-loom-start-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-has-no-own-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: acceptance-row-overview-replace-contradicts-d6
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-bound-incomplete-after-d5-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-miscounts-focus-users
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-degraded-click-undecided
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
The one gating finding is D2's re-arm bound, which treats every `lyx loom start` as an explicit run start although it is also the resume verb.
Its fix changes a decision, so the new text needs a fresh review.
The other five are local corrections whose text the review fixes: a missing acceptance row for D4, a contradicting acceptance row against D6, D4's incomplete bound, D3's short list of `Focus` users, and D7's undecided degraded click.
