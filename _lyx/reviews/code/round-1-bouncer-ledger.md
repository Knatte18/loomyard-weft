---
round: 1
ledger:
  - key: unlicensed-verifytree-flake-fix
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: fingerprint-neighbour-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
F1 (`unlicensed-verifytree-flake-fix`) is a scope finding about the `stoppableCommand` flake fix outside the task's scope.
F2 (`fingerprint-neighbour-row-missing`) is a design finding asking for a fingerprint test row that pins a lens-named neighbour as still hashed.
Neither gates.
