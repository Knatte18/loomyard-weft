---
round: 2
ledger:
  - key: embed-own-test-files-invisible
    rounds: [1]
    status: resolved
  - key: spawn-marked-selection-unrecorded
    rounds: [1]
    status: resolved
  - key: method-body-row-missing
    rounds: [1]
    status: resolved
  - key: d5-cases-without-row
    rounds: [1]
    status: resolved
  - key: embed-pattern-shapes-unpinned
    rounds: [1]
    status: resolved
  - key: deleted-file-imports-miss
    rounds: [1]
    status: resolved
  - key: dotless-import-treated-stdlib
    rounds: [1]
    status: resolved
  - key: doc-comments-lean-on-symbols
    rounds: [1]
    status: resolved
  - key: linked-test-file-embed-unpinned
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, five at MEDIUM and three at LOW.
Round 2's review rejected each of them in its `## Rejected` section as handled, so all eight are resolved.
Round 2 raised one new LOW finding, F1: no row pins that a linked package's test files are not embed sources.
Its fix is a single test row whose result the finding text already fixes.
