---
round: 1
ledger:
  - key: typed-references-partial-ast-checked
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: verify-nested-module-missing-test-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: importsof-import-name-missing-test-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised three LOW findings, none gating.
F1 maps to `typed-references-partial-ast-checked`: a file with a parse error is marked checked from its partial AST.
F2 maps to `verify-nested-module-missing-test-rows`, and F3 to `importsof-import-name-missing-test-rows`; both are test-row gaps.
There was no previous ledger.
