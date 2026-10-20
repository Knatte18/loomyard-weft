---
round: 2
ledger:
  - key: old-binary-reseed-reverts-fans
    rounds: [1]
    status: resolved
  - key: template-edits-stop-reaching-hub
    rounds: [1]
    status: resolved
  - key: template-lens-fan-cannot-be-deleted
    rounds: [1]
    status: resolved
  - key: set-on-any-name-unbounded
    rounds: [1]
    status: resolved
  - key: hub-only-entries-warned-unread
    rounds: [1]
    status: resolved
  - key: expansion-bound-wrong-exposure
    rounds: [1]
    status: resolved
  - key: deletion-staging-already-settled
    rounds: [1]
    status: resolved
  - key: missing-acceptance-criteria
    rounds: [1]
    status: resolved
  - key: kept-target-line-renders-whole-mappings
    rounds: [1]
    status: resolved
  - key: prime-copy-start-drops-burler-mappings
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: set-value-shape-and-deep-paths-unstated
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1's nine findings are resolved: the round 2 review reached each of its four focus directives and raised none of those keys again.
Round 2 raised two new findings.
The first is D4's prime-copy start contradicting its own no-overwrite bound, which opens a loss window for burler's mappings.
The second is the unstated value shape and deep-path refusal for `--set` under the per-name kind.
Both are local text corrections whose result the finding text fixes.
