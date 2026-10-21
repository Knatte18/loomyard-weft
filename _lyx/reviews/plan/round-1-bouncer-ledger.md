---
round: 1
ledger:
  - key: new-keys-break-exact-set-pins-and-docs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: subjectless-bouncer-writes-generation-record
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: rework-record-carry-skipped-on-restep
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-in-patch-id-admission-unbounded
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: case3-reject-comparison-no-settle-time
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: goto-guard-refusal-promises-amendment-for-merge-review
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: batten-budget-clause-placeholder-run-id
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: goto-refusal-way-forward-omits-resume
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: loom-budget-way-forward-placeholder-signature
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 9 findings: F1 to F9 map in order to the keys above.
F1 (BLOCKING, design) is a missed-edit gap: new config and envelope keys break exact-set test pins and key-list docs no card lists.
F3 (MEDIUM, design) is a sequencing flaw in the PR-Rework record carry on a re-step.
Both gate.
F2 and F5 change card contracts, so their fixes need fresh review.
The rest are local text corrections.
