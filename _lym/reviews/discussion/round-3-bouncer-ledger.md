---
round: 3
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
    rounds: [1, 2, 3]
    status: resolved
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
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
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
    status: resolved
  - key: linked-package-without-covered-block-uncompared
    rounds: [2]
    status: resolved
  - key: non-body-miss-hit-rate-unmeasured
    rounds: [2]
    status: resolved
  - key: signature-receiver-change-unselected
    rounds: [2]
    status: resolved
  - key: go-directives-treated-as-comments
    rounds: [2]
    status: resolved
  - key: selected-invocation-can-run-zero-tests
    rounds: [2, 3]
    status: resolved
  - key: unparsable-covered-file-outcome-missing
    rounds: [2]
    status: resolved
  - key: line-ending-conversion-misses-everything
    rounds: [2]
    status: resolved
  - key: gate-slot-constraint-misstates-map-read
    rounds: [2]
    status: resolved
  - key: union-names-test-no-host-binary-compiles
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dirty-recheck-not-fresh-read-window-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: inert-paths-exempts-embedded-file
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: predeclared-name-added-rebinds-unchanged-code
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised sixteen findings and round 2 eleven; round 3 raises five, a falling count.
Round 3 rejects as not defects the round-2 themes of embedded subdirectory files, linked packages without a covered block, the non-body hit rate, signature and receiver changes, and `//go:` directives, so those keys are resolved.
The round-2 zero-test key is resolved on its named routes; round 3's F2 is a separate route (build-constrained test files) under a new key.
The round-2 dirty-check key is resolved on its defect (own directory only); F3 is a different defect (freshness of the recheck and an unbounded race window).

`testmap-upkeep-question-misstates-note` continues as F1, now `consistency`: the parent's Added after Discussion entries answered the question, and the record does not yet apply them.
