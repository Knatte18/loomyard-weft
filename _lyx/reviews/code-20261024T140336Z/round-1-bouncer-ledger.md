---
round: 1
ledger:
  - key: ceiling-env-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: relative-entry-claim-unasserted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-isolation-background-omits-ceiling-var
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, all local corrections.
F1 asks for a serial test pinning the `GIT_CEILING_DIRECTORIES` read in `RepositoryRootOf`.
F2 asks for an assertion that the relative entry is dropped.
F3 asks for the background file to list the new variable.
