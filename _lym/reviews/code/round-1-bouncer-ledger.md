---
round: 1
ledger:
  - key: claude-local-instruction-name-redeclared
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: heddle-file-target-engine-literals
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: codex-drops-omitted-hook-without-plugin
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: parity-engine-names-hardcoded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: report-tempdir-removal-untested-on-errors
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: manifest-fields-unrun-and-verdict-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: manifest-report-of-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: production-passage-reader-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: claude-resolution-failure-comment-overclaims
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: claude-manifest-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: manifest-write-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shipped-layer-test-missing-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-verb-list-omits-manifest
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised thirteen findings: F1 to F13, in the order of the keys above.
Two are BLOCKING and are both `PATTERN-code-conventions` literal duplications.
Five are gating design findings at MEDIUM: an omitted-hook gap in the Codex manifest and four missing tests.
F4 is a MEDIUM decision the author must record or resolve.
The rest are LOW or NIT and mechanical.
There was no earlier ledger, so nothing carries forward.
