---
round: 1
ledger:
  - key: targets-refusal-missing-role-clause
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: runtest-dotnet-warning-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: namesproject-built-assembly-reaches-all
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: namesproject-basename-only-match
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runwhole-notice-before-later-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unrunprojectlines-missing-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6 in that order), none of them carried in from an earlier round.
F2 and F3 are the gating ones: the `runTest` warning wiring has no test, and `namesProject` lets a command naming only a built assembly count as reaching every project.
The rest are a missing role clause on the targets refusal, file-name-only project matching, a run-whole notice printed before a later refusal, and missing unit rows.
