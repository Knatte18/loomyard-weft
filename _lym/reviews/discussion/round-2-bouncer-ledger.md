---
round: 2
ledger:
  - key: map-freshness-against-tier-base
    rounds: [1]
    status: resolved
  - key: importing-package-own-files-unchecked
    rounds: [1]
    status: resolved
  - key: spawn-scan-blind-spots-unbounded
    rounds: [1]
    status: resolved
  - key: testmap-dirty-check-own-directory-only
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: format-marker-survives-older-binary-rewrite
    rounds: [1]
    status: resolved
  - key: fabric-boundary-constraint-wrong-path
    rounds: [1]
    status: resolved
  - key: gate-yaml-question-is-writer-choice
    rounds: [1]
    status: resolved
  - key: testmap-upkeep-question-misstates-note
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: tier2-share-unmeasured
    rounds: [1]
    status: resolved
  - key: named-import-change-alters-init-set
    rounds: [1]
    status: resolved
  - key: same-named-init-pairing-undefined
    rounds: [1]
    status: resolved
  - key: d3-criterion-contradicts-d6-over-budget
    rounds: [1]
    status: resolved
  - key: d7-missing-whole-run-labels
    rounds: [1]
    status: resolved
  - key: d5-new-file-criterion-ambiguous
    rounds: [1]
    status: resolved
  - key: packageblocks-hint-wrong-data
    rounds: [1]
    status: resolved
  - key: test-speed-constraint-restates-pattern
    rounds: [1]
    status: resolved
  - key: embedded-subdir-files-not-recorded
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: linked-package-without-covered-block-uncompared
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: non-body-miss-hit-rate-unmeasured
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: signature-receiver-change-unselected
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: go-directives-treated-as-comments
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: selected-invocation-can-run-zero-tests
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unparsable-covered-file-outcome-missing
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: line-ending-conversion-misses-everything
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: gate-slot-constraint-misstates-map-read
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings; the round-2 review re-raises none of them except two that map onto round-2 findings.
`testmap-dirty-check-own-directory-only` continues as F10 (mid-walk edit under D2's widened covered set).
`testmap-upkeep-question-misstates-note` continues as F4, now a `decision` question for the parent about D2's staleness change against note `testmap-upkeep`.

Round 2 opens three BLOCKING design gaps in the comparison set: embedded files in covered subdirectories (F1), linked packages without a covered block (F2), and the unmeasured hit rate of the non-body miss (F3).
Three MEDIUM design gaps follow: signature or receiver changes (F5), `//go:` directives (F6), and the zero-test selected invocation (F7).
The remaining LOW findings are local corrections or bounds to state.
