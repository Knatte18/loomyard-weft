---
round: 1
ledger:
  - key: fan-shape-error-no-way-forward-or-spec-row
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: bouncer-cluster-fans-ignores-recipe-fan
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: twill-engine-refusal-two-ways-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unrecorded-names-refusal-archives-before-probe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: single-fixer-keeps-stale-twill-lenses-and-forks
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 (F1 to F5) raised five findings against cards 3, 5, 6, 7 and 8.
F1 is a missing way forward and refusal-spec row for `fanShapeError` in card 8.
F2 is the Bouncer's raw `ClusterFans` list overriding the recipe fan across cards 6 and 7.
F3 is a doubled way-forward clause in card 3's twill engine refusal, and F4 is card 5's archive-before-probe ordering in `BurlerFixProducer.Call`.
F5 is `fixedExit` leaving stale twill lenses and fork count in the record.
