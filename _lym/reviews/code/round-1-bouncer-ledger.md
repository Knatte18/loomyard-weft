---
round: 1
ledger:
  - key: prefix-over-empty-pair-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pairtaskbranch-rules-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: branch-to-slug-steps-untested-with-prefix
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: remove-origin-lookup-launchers-and-double-ask
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: origin-only-pair-named-with-current-prefix
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pairtaskbranch-errors-on-unresolvable-records-root
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pr-lookup-and-readopt-prefix-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: branch-prefix-template-description-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-ownedmanagedbranch-signature-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remoteteardown-field-comment-covers-other-field
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so every key is new and open.
Keys map to findings as follows: F1 prefix-over-empty-pair-untested, F2 pairtaskbranch-rules-untested, F3 branch-to-slug-steps-untested-with-prefix, F4 remove-origin-lookup-launchers-and-double-ask, F5 origin-only-pair-named-with-current-prefix, F6 pairtaskbranch-errors-on-unresolvable-records-root, F7 pr-lookup-and-readopt-prefix-untested, F8 branch-prefix-template-description-stale, F9 stale-ownedmanagedbranch-signature-comment, F10 remoteteardown-field-comment-covers-other-field.
F1 to F3 gate at MEDIUM and are test gaps for a changed prefix over a live bare-slug pair.
F5 adds a Topology method, so its fix is new design and needs a fresh review.
