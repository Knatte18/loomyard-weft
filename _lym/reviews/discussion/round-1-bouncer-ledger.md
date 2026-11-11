---
round: 1
ledger:
  - key: map-freshness-against-tier-base
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: importing-package-own-files-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spawn-scan-blind-spots-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: testmap-dirty-check-own-directory-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: format-marker-survives-older-binary-rewrite
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fabric-boundary-constraint-wrong-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gate-yaml-question-is-writer-choice
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: testmap-upkeep-question-misstates-note
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tier2-share-unmeasured
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: named-import-change-alters-init-set
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: same-named-init-pairing-undefined
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d3-criterion-contradicts-d6-over-budget
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-missing-whole-run-labels
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-new-file-criterion-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: packageblocks-hint-wrong-data
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-speed-constraint-restates-pattern
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The gating finding is the BLOCKING design flaw in how D5 checks map freshness against the tier's base, with design-class MEDIUMs on D2's covered-package checks, D3's spawn-scan bounds and the fabric-boundary constraint.
Every other finding is a local correction.
