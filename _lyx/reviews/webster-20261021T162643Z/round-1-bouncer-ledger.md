---
round: 1
ledger:
  - key: undeclared-pattern-entry-edits
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: clone-marker-rollback-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 (`undeclared-pattern-entry-edits`): a commit dropped clauses from ten PATTERN entries the declared changes do not list; the fix restores them from the merge base.
F2 (`clone-marker-rollback-untested`): the clone's marker cleanup on a failed board commit has no test.
