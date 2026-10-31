---
round: 1
ledger:
  - key: teardown-resume-pair-present-reconcile-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-owner-machine-stale-on-teardown-skip
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: battenshed-doc-stale-on-resume-preflight
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: teardown-skip-unowned-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: owner-test-file-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), in order the keys above.
F1 is the only gating one: the Worktree-Teardown owner pre-flight test lacks a pair-present row that still expects the reconcile.
F2 and F3 are stale descriptions of the resume reconcile in `pattern/PATTERN-owner-machine.md` and `internal/battenshed/doc.go`.
F4 asks for an unowned row on the skip path, and F5 for an updated header in `internal/battencli/owner_test.go`.
All fixes are local corrections whose text the findings fix.
