---
round: 1
ledger:
  - key: sparse-hub-base-entry-over-line-cap
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: undeclared-pattern-entries-shortened
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: runwithenv-extraenv-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: duplicate-skip-push-variable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Two gating findings are open: the declared `PATTERN-sparse-hub-base` line exceeds the entry-line cap, and eleven undeclared entries lost clauses.
Two LOW findings are open: `RunWithEnv` lacks a test of its `extraEnv` path, and the push-skip variable is duplicated across test tiers.
