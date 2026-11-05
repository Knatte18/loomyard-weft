---
round: 2
ledger:
  - key: prefix-over-empty-pair-untested
    rounds: [1]
    status: resolved
  - key: pairtaskbranch-rules-untested
    rounds: [1]
    status: resolved
  - key: branch-to-slug-steps-untested-with-prefix
    rounds: [1]
    status: resolved
  - key: remove-origin-lookup-launchers-and-double-ask
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: origin-only-pair-named-with-current-prefix
    rounds: [1]
    status: resolved
  - key: pairtaskbranch-errors-on-unresolvable-records-root
    rounds: [1]
    status: resolved
  - key: pr-lookup-and-readopt-prefix-untested
    rounds: [1]
    status: resolved
  - key: branch-prefix-template-description-stale
    rounds: [1]
    status: resolved
  - key: stale-ownedmanagedbranch-signature-comment
    rounds: [1]
    status: resolved
  - key: remoteteardown-field-comment-covers-other-field
    rounds: [1]
    status: resolved
  - key: detached-literal-survives-next-to-constant
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: prefix-changed-between-nonempty-values-unrecognised
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: unresolvable-records-root-skip-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1's ten findings did not recur in round 2; the nine that did not reappear are resolved.
The launcher-only origin lookup key reopens as round 2's F3 at LOW, with its own fix text.
Round 2 adds three keys: F1 a surviving `"(detached)"` literal in `originreconcile.go` next to the `detachedBranch` constant (BLOCKING), F2 a `branch_prefix` change between two non-empty values leaving live pairs unrecognised (decision, deferred to the parent), and F4 the missing test for the records worktree whose repository root cannot be resolved.
