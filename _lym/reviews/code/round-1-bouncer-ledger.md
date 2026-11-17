---
round: 1
ledger:
  - key: record-edit-failed-apply-patch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overview-codex-edit-hook-missing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: patched-files-leading-whitespace
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: record-edit-append-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: child-thread-id-fallbacks-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: file-edit-signal-empty-path-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: hook-edit-docs-one-record
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, F1 to F7, in the order of the ledger above.
F1 is the only gating one: a failed Codex `apply_patch` may be recorded as an edit.
The rest are a doc gap in the overview, a header-parsing edge, three test gaps and a doc wording fix.
