---
round: 1
ledger:
  - key: template-tests-lack-keep-markers
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: piece-cap-bytes-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stencil-overrides-rubric-range
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: staged-patch-not-committed-range
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: split-order-and-assistant-pieces-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: missing-rows-failure-label-error-paths
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: split-file-continuation-no-header
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: package-parse-space-b-slash
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: piece-file-name-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overcap-line-cut-splits-rune
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: stale-doc-fills-with-all-markers
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings, all open.
F1 is BLOCKING: two kept template tests lack the `//testtiming:keep` marker `PATTERN-test-speed` requires.
F2 to F5 are MEDIUM design findings: the byte-only piece cap, the stencil overriding the rubric's range, the staged patch reading the working tree instead of the committed range, and unpinned split order and assistant pieces.
F6 to F11 are LOW or NIT: missing test rows, headerless continuation pieces, a package parse misreading a path, an unbounded piece file name, a rune split and a stale test doc.
F4 is the one fix that adds a new seam and interface, so it needs a fresh review.
